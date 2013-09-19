---
layout: post
title: "AOP 101: Speeding up Spring's JavaMailSenderImpl with AOP"
date: 2008-11-15 18:40:35
comments: true
categories: [Chapter 08 - Communicating, Tutorials]
---
<div class="alert info">This material is based on an earlier version of <a href="http://www.manning.com/wheeler/">Spring in Practice, chapter 8</a>&mdash;one that predates the <code>@Async</code> annotation. Nowadays I would recommend using <code>@Async</code> and the Spring Task Execution API for making JavaMail calls asynchonous. The book covers the newer approach. The material in this post is still useful for understanding what you can do with AOP though.</div>

In this article we'll learn how we can speed up Spring's <code>JavaMailSenderImpl</code> with some thread-forking AOP. Though we're using JavaMail as an example, this tutorial should be useful to people looking for a code-based introduction to Spring's support for AOP. Note at the outset that I don't really go into AOP concepts and terminology, but I do show some simple code that you should be able to follow if you already know the basic concepts and just want to see what the code looks like.

There are lots of situations in which we want our application to send out an automated e-mail. You might for instance want to send a confirmation e-mail in response to new user registrations or mailing list subscriptions and unsubscriptions. In Spring this probably means that you would use one of the various <code>JavaMailSenderImpl.send()</code> methods. Here's a sample <code>applicationContext.xml</code> file.

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-2.5.xsd&gt;
    
    &lt;jee:jndi-lookup id="mailSession" jndi-name="mail/Session" resource-ref="true"/&gt;
    
    &lt;bean id="mailSender"
        class="org.springframework.mail.javamail.JavaMailSenderImpl"
        p:session-ref="mailSession"/&gt;
    
    &lt;bean id="mailingListService"
        class="app.service.MailingListServiceImpl"
        p:mailSender-ref="mailSender"/&gt;
        
    ...
    
&lt;/beans&gt;</pre>

Here's how it looks from the Java side:

<pre>package app.service;

... imports ...

public class MailingListServiceImpl implements MailingListService {
    private JavaMailSender mailSender;
    
    public void setMailSender(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }
    
    private void sendConfirmSubscriptionEmail(Subscriber subscriber) {
        MimeMessage message = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message);
        
        String text = ...
        
        try {
            helper.setSubject("Please confirm your subscription");
            helper.setTo(subscriber.getEmail());
            helper.setFrom(noReplyEmailAddress);
            helper.setSentDate(subscriber.getDateCreated());
            helper.setText(text, true);
        } catch (MessagingException e) {
            throw new RuntimeException(e);
        }
        
        mailSender.send(message);
    }
    
    ...
}</pre>

(For more information on Spring/JavaMail integration, please see my article <a href="#">Send E-mail Using Spring and JavaMail</a>.)

This works fine, but one thing your end users might notice is a fairly significant delay while <code>JavaMailSenderImpl.send()</code> does whatever it's doing to send your e-mail (presumably <a href="http://springinpractice.com/2008/05/05/smtp-and-smtp-auth/">negotiating with the SMTP server and sending the actual e-mail</a>). While the delay probably isn't large enough to provoke rioting in the streets, it's certainly noticeable, and in many use cases it's unnecessary. E-mail is itself an asynchronous communications medium, so unless there's an important reason to let the end user know about errors that may occur while trying to send the e-mail (and there may be), one option you might consider is making the <code>send()</code> call on a separate thread.

Now let's look at a few different ways to do that.

<h3>Method 1: Spawn a new thread manually</h3>

One possibility would be to spawn a new thread manually whenever you want to call <code>JavaMailSenderImpl.send()</code>. In other words, you implement the <code>Runnable</code> interface with a call to <code>send()</code>, you pass it into a <code>Thread</code>, and then you start the thread.

This technique has some advantages. It's conceptually straightforward. Also it's easy to be selective about the cases in which you do and don't want to fork. Again, there may well be times where you want the end user to know if the <code>send()</code> call generated an exception, and if that's true, then you simply refrain from forking the thread.

If you're not careful, the approach can lead to widespread violation of the <a href="http://en.wikipedia.org/wiki/Don%27t_repeat_yourself">DRY principle</a>. You might end up rewriting the same thread-forking code every time you send an e-mail. You can of course control this by creating one or more utility methods to send an e-mail on a separate thread, and that is a good approach.

One drawback with this approach, though, is that it may be either inconvenient or else a non-option. If you have an existing app with lots of calls to create e-mail, then you'd need to update all the instances of that code with the new code. In most cases that's probably doable though it may be inconvenient. But it may be that you're not in a position to change the client code. (Maybe it's a third-party library, for instance.) The client code calls an injected <code>JavaMailSender</code> instance, say, and that's the way it is. In that event you'll want to consider one of the two following alternative methods.

<h3>Method 2: Create a JavaMailSender wrapper</h3>

Another method would be to implement a <code>JavaMailSender</code> wrapper. (<code>JavaMailSender</code> is of course the interface to which <code>JavaMailSenderImpl</code> conforms.) The <code>JavaMailSender</code> interface has six different <code>send()</code> methods (<a href="http://static.springframework.org/spring/docs/2.5.6/api/org/springframework/mail/javamail/JavaMailSender.html">here are the Javadocs</a>), and so you can just implement the thread-forking code for each of the six methods. (Probably each method would create a <code>Runnable</code> and then pass that to a thread-forking utility method.) Then you inject your wrapper into your service beans instead of injecting the <code>JavaMailSenderImpl</code> bean directly.

This approach is pretty good. It's still straightforward, and it allows you to avoid violating DRY. Also, because it's entirely transparent to client code, it can deal with cases in which you either can't or else don't want to modify said client code.

One possible challenge is that you may find it a little tough to exercise fine-grained control over the cases in which you use the wrapper and the cases in which you don't. If it's important for your code to exercise that kind of control, then arguably it would be reasonable to associate the forking/non-forking semantics injected <code>JavaMailSender</code> beans. You might for example inject two <code>JavaMailSender</code> instances into the service bean&mdash;one forking and one non-forking.

A minor grumble about the wrapper method is that it ties the thread-forking behavior to specific interfaces, such as <code>JavaMailSender</code>. That's not too big a deal in this particular case, since it's not such a problem to spawn a new thread. But if you have other cases where you decide you want to create a new thread, you might decide that you'd like to factor thread-forking out as a separate behavior and be able to apply that in multiple contexts.

So let's see how to do that using Spring's support for AspectJ-flavored AOP.

<h3>Method 3: Use AOP to wrap JavaMailSenderImpl.send()</h3>

This is a fun and elegant method. Even though this article is called "AOP 101", I'm not planning to explain the concepts or weird terminology; rather I just want to show you the code and assume that you'll be able to see what's happening.

First we need to create an "advice" class. This is the code that we're going to wrap around our <code>send()</code> invocations.

<pre>package app.aop;

import org.apache.log4j.Logger;
import org.aspectj.lang.ProceedingJoinPoint;

public class ForkAdvice {
    private static final Logger log = Logger.getLogger(ForkAdvice.class);
    
    public void fork(final ProceedingJoinPoint pjp) {
        new Thread(new Runnable() {
            public void run() {
                log.info("Forking method execution: " + pjp);
                try {
                    pjp.proceed();
                } catch (Throwable t) {
                    // All we can do is log the error.
                    log.error(t);
                }
            }
        }).start();
    }
}</pre>

The fork method is "around" advice that we're going to use to advise our calls to <code>JavaMailSenderImpl.send()</code>. As you can see, it creates a new thread and starts it. In the <code>run()</code> body, we simply execute the advised method by calling <code>pjp.proceed()</code>.

As an aside, the <code>ProceedingJoinPoint</code> class is provided by the AspectJ class library, but note that we're not using full-blown AspectJ here&mdash;we're in fact using Spring AOP. Full AspectJ involves a special aspect language and compiler to generate classes with the advice woven into the class bytecode itself. Spring AOP on the other hand uses dynamic proxies (either the interface variety that comes with Java, or else class proxies via CGLIB) to advise classes. While Spring AOP borrows classes and also the AspectJ pointcut language from AspectJ, its use of dynamic (runtime) proxies as opposed to bytecode-level advice integration distinguishes it from AspectJ.

Now it's time to update our application context with our AOP configuration.

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-2.5.xsd&gt;
    
    &lt;jee:jndi-lookup id="mailSession" jndi-name="mail/Session" resource-ref="true"/&gt;
    
    &lt;bean id="mailSender"
        class="org.springframework.mail.javamail.JavaMailSenderImpl"
        p:session-ref="mailSession"/&gt;
    
    &lt;bean id="mailingListService"
        class="app.service.MailingListServiceImpl"
        p:mailSender-ref="mailSender"/&gt;
        
    &lt;bean id="forkAdvice" class="app.aop.ForkAdvice"/&gt;
    
    &lt;aop:config&gt;
        &lt;aop:aspect ref="forkAdvice"&gt;
            &lt;aop:around method="fork"
pointcut="execution(* org.springframework.mail.javamail.JavaMailSenderImpl.send(..))"/&gt;
        &lt;/aop:aspect&gt;
    &lt;/aop:config&gt;
    
    ...
    
&lt;/beans&gt;</pre>

This is similar to what we had before, but there are a couple of differences. First, note that we've declared the <code>aop</code> namespace here. That of course allows us to use the namespace configuration feature that Spring 2.0 introduced. The other change is that we've added a definition for our advice bean as well as some AOP configuration. In <code>aop:aspect</code> we point to our <code>forkAdvice</code> as the advising class to be applied, we indicate that it will be "around" advice, we specify the advising method, and finally we specify a pointcut that indicates which method calls will be advised/wrapped. We use the <a href="http://www.eclipse.org/aspectj/doc/released/progguide/semantics-pointcuts.html">AspectJ pointcut language</a> to specify a pointcut. Here we're indicating that we want all calls to any of the <code>JavaMailSenderImpl.send()</code> methods to be advised.

As mentioned previously, this technique is like the wrapper technique in that you can use it to add the forking behavior in a way that's transparent to client code. Moreover you can use it not just for JavaMail but really for any method where you want to create a new thread before executing the method. You just add the appropriate <code>aop:around</code> definitions to the <code>aop:aspect</code> definition and you're in business.

<div class="endnote">Post migrated from my Wheeler Software site.</div>
