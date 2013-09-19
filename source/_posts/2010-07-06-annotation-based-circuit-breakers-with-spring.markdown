---
layout: post
title: "Annotation-based circuit breakers with Spring"
date: 2010-07-06 22:10:11
comments: true
categories: [Chapter 14 - Site-up, Tutorials]
---
<h3>The punch line</h3>

First, here's the punch line for readers who already know what a circuit breaker is:

<pre style="margin:20px 0;">
@Service
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.DEFAULT,
    readOnly = true)
public class MessageServiceImpl implements MessageService {
    @Inject private MessageDao messageDao;

    @GuardedByCircuitBreaker("messageServiceBreaker")
    public Message getMotd() {
        return messageDao.getMotd();
    }

    @GuardedByCircuitBreaker("messageServiceBreaker")
    public List getImportantMessages() {
        return messageDao.getImportantMessages();
    }
}
</pre>

If you find that exciting (or at least interesting, if you don't know about circuit breakers yet), read on...

<h3>A proper introduction</h3>

One of my favorite books is Michael Nygard's <a href="http://www.pragprog.com/titles/mnee/release-it">Release It!</a> (Pragmatic). The idea behind the book is that when we build apps, the feature set isn't the only thing that matters. We need to pay attention to app vulnerabilities that too often result in performance issues or else downtime. A central thesis is that plenty of vulnerabilities are sufficiently common to warrant being classified as antipatterns, and there are corresponding patterns of correct engineering practice to address the antipatterns.

Let's look at one of Nygard's antipatterns, the cascading failure.

<h3>Cascading failures</h3>

Probably everybody has seen cascading failures. It's essentially vertical fault propagation. The system has a fault somewhere low in the stack and the fault travels up the stack in an uncontrolled fashion until the outage is much worse than it should have been. Consider for instance the following system:

<img src="http://springinpractice.s3.amazonaws.com/blog/images/portal.png" alt="Portal" />

In this system we have a portal home page that depends upon a web service to deliver important messages to end users. The web service in turn pulls messages out of a database. Standard stuff.

Now suppose there's some kind of problem with the database. Maybe a missing index leads to a bad execution plan, which leads to slow query times, leading to the web service's database calls queuing up and exhausting threads, causing the portal's calls to the web service to do the same thing, causing the browser's calls to the portal to do the same. Here's what that looks like:

<img src="http://springinpractice.s3.amazonaws.com/blog/images/portal-cascade.png" alt="Broken portal" />

While it goes without saying that the important messages are important, it's doubtful that they're so important that a missing index should be allowed to create complete system unavailability. (In this example, we took out a portal home page, so nobody can get in. If users are using other apps in the portal then they can continue to use them, but nobody new can enter.)

Not good. But probably pretty common. I know I've seen my fair share.

<h3>Circuit breakers to the rescue</h3>

A nice way to deal with cascading failures is to engineer the integration points to stop propagating failure. The pattern is called the circuit breaker, and it's modeled after physical circuit breakers in the real world. We install a circuit breaker at each integration point we want to treat. Under normal conditions, requests and responses flow over the breaker just like current flows over a real circuit breaker. But when the breaker detects a problem with the service at the far end of the breaker, it trips, thus preventing requests and responses from crossing over the integration point, at least until the breaker is reset.

The circuit breaker pattern has two major benefits:

<ul>
    <li><b>Breakers protect the client:</b> We don't want breakers wasting valuable resources (e.g., threads, network connections, their associated physical resources) calling services that are known to be down. Especially since the antipattern normally involves the thread just sitting there waiting for a response. The breaker prevents this by failing fast.</li>
    <li><b>Breakers protect the service:</b> Sometimes the service is bogging down just due to insufficient capacity. In that situation, breakers prevent a capacity issue from becoming complete service unavailability by throttling load to the service. (There's still an availability issue here, but it's partial.)</li>
</ul>

That's the concept. From a technical perspective, a breaker is a state machine with three states:

<ul>
    <li><b>Closed:</b> The breaker's normal state. Requests and responses flow freely across the integration point.</li>
    <li><b>Open:</b> The breaker's state when there's a problem. No requests can make it across the integration point. In effect, problems with the service are contained, and no propagation occurs.</li>
    <li><b>Half-open:</b> The breaker's state when it decides it's time to "try again". The breaker doesn't stay open forever. Periodically it will allow a single request through to see whether the service is working again. If so, the breaker resets (goes closed). Otherwise the breaker trips again (goes open).</li>
</ul>

Here's a state transition diagram for a circuit breaker. This is adapted from a diagram in Nygard's book:

<img src="http://kite.s3.amazonaws.com/wiki/images/circuit-breaker-state-transition.png" alt="Circuit breaker state transition diagram" />

From the diagram we can see that the trip condition is a certain number of consecutive exceptions. In principle we could use whatever trip condition we like. For example, it could be too many exceptions within a specified time window (i.e. excessive exception rate), too many SLA misses, or perhaps something else. In the implementation we'll describe below it's too many consecutive exceptions of specific types. In general we want to filter out exceptions that the user can intentionally generate (for example, by manipulating HTTP parameters or path variables in a URL) because we don't want to create a vector for DoS attacks.

The other thing the diagram reveals is a timeout parameter. This is just the amount of time the breaker should remain open before going half-open and trying the service again.

That's how breakers work in the abstract. Now let's look at a specific implementation based on Spring.

<h3>My circuit breaker implementation</h3>

My circuit breaker implementation is part of a nascent open source effort called <a href="http://code.google.com/p/kite-lib/">Kite</a> that I started a few months back. I haven't done terribly much with the project yet, mostly because I'm trying to get the book wrapped up, but once I get above water again I'll be returning to the effort. Fortunately though I was able to get a fairly robust circuit breaker implementation out there with some useful features.

The breaker is based on Spring, and that being the case, it's code-level design is very Spring-like. It's pretty similar to transaction management in Spring. You configure some doodads in the application context file and then you have the choice between programmatic application of breakers (via templates; again, very similar to other templates like JdbcTemplate) or else declarative configuration. The declarative approach further supports XML-based configuration and annotation-based configuration.

Let's look at each in turn.

<h4>Programmatic circuit breakers</h4>

This will be familiar to anybody who has used HibernateTemplate, JdbcTemplate, TransactionTemplate, etc. The following is an example of what a Spring Web MVC controller might look like when using programmatic circuit breakers:

<pre style="margin:20px 0;">
@Inject MessageService messageService;
@Inject CircuitBreakerTemplate breaker;
...

public String getMotd() {
    ...
    try {
        Message motd = breaker.execute(new CircuitBreakerCallback() {
            public Message doInCircuitBreaker() throws Exception {
                return messageService.getMotd();
            }
        });
        model.addAttribute("motd", motd);
    } catch (Exception e) {
        log.error("Couldn't get MOTD: {}", e.getMessage());
    }
    ...
}
</pre>

While it's nice to have the flexibility to go programmatic, usually we don't want to go through all that trouble. The next pair of approaches simplifies things quite a bit.

<h4>Declarative circuit breakers using XML</h4>

Spring of course offers XML-based configuration for its various frameworks, and since Spring 2.0, Spring has supported custom namespaces that effectively let framework designers design a domain-specific language for framework configuration. That feature is too cool to pass up, so I built that into my implementation. Here's what the app context config looks like:

<pre style="margin:20px 0;">
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans:beans xmlns="http://springinpractice.com/schema/kite"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://springinpractice.com/schema/kite
        http:// ... kite schema location ... "&gt;

    &lt;context:mbean-export /&gt;
    
    &lt;circuit-breaker
        id="messageServiceBreaker"
        exceptionThreshold="3"
        timeout="30000" /&gt;

    &lt;circuit-breaker-advice
        id="messageServiceBreakerAdvice"
        breaker="messageServiceBreaker" /&gt;

    &lt;aop:config&gt;
        &lt;aop:pointcut id="messageServicePointcut"
            expression="execution(* kite.sample02.service.MessageServiceImpl.*(..))" /&gt;
        &lt;aop:advisor advice-ref="messageServiceBreakerAdvice"
            pointcut-ref="messageServicePointcut"
            order="0" /&gt;
    &lt;/aop:config&gt;
&lt;/beans:beans&gt;
</pre>

In this configuration you can see that I've made <code>kite</code> the default namespace, and then I've done a bunch of config that's hopefully mostly self-explanatory, if you know AOP anyway. (If not, that's fine. Read on.) The <code>&lt;context:mbean-export /&gt;</code> is there because the breaker implementation is JMX-enabled so your NOC or support staff (maybe it's you) can manually trip and reset the breaker as needed. That's a must-have feature in my opinion. Then I define a breaker, an AOP advice for the breaker, and then a pointcut and advisor to apply the breaker to the desired location.

This is a nice approach, especially if you're not a big fan of annotations, which a lot of people aren't. I however like annotations quite a bit, so I went ahead and implemented support for annotation-based configuration too, as described below.

<h4>Declarative circuit breakers using annotations</h4>

Here's the app context:

<pre style="margin:20px 0;">
&lt;beans:beans xmlns="http://springinpractice.com/schema/kite"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://springinpractice.com/schema/kite
        http:// ... kite schema location ... "&gt;

    &lt;context:mbean-export /&gt;

    &lt;circuit-breaker
        id="messageServiceBreaker"
        exceptionThreshold="3"
        timeout="30000" /&gt;
    
    &lt;annotation-config order="0" /&gt;
&lt;/beans:beans&gt;
</pre>

And here's the annotated service bean:

<pre style="margin:20px 0;">
package kite.sample03.service;

import java.util.List;
import javax.inject.Inject;
import kite.annotation.GuardedByCircuitBreaker;
import kite.sample03.dao.MessageDao;
import kite.sample03.model.Message;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.DEFAULT,
    readOnly = true)
public class MessageServiceImpl implements MessageService {
    @Inject private MessageDao messageDao;

    @GuardedByCircuitBreaker("messageServiceBreaker")
    public Message getMotd() {
        return messageDao.getMotd();
    }

    @GuardedByCircuitBreaker("messageServiceBreaker")
    public List getImportantMessages() {
        return messageDao.getImportantMessages();
    }
}
</pre>

Nice and clean Spring-style annotation-based configuration.

<h3>Conclusion</h3>

Hope you enjoyed the post. If you want to give it a try, go to the <a href="http://code.google.com/p/kite-lib/">Kite project</a> and try it out. Also, chapter 14 in my book <a href="http://www.manning.com/wheeler/">Spring in Practice</a> shows how to go about implementing your own Spring-style framework code, should you get the itch, and it's based on the circuit breaker implementation we just described. It explains how to implement a template, a custom namespace, JMX support, AOP-based configuration and also annotation-based configuration.
