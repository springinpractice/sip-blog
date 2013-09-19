---
link: http://springinpractice.com/2008/12/02/new-stuff-in-spring-30/
layout: post
title: New stuff in Spring 3.0
date: 2008-12-02 19:59:26
comments: true
categories: [News]
---
This morning I attended a talk by Juergen Hoeller on what's coming in Spring 3.0. First, he said that we can expect the first milestone release sometime this week, and said that he expected there to be about three milestone releases I think. RC1 is supposed to be out around March 2009 and final around April 2009.

Echoing something I've heard Keith Donald say, Juergen said that Spring 3.0 is intended to complete the work that was started in Spring 2.5. Keith described the release as being "evolutionary, not revolutionary." That may be, but there's still quite a bit to be excited about.  Here's what I got from the talk today. (Hold onto your hats because I took good notes.)
<h3>Java and JEE compatibility</h3>
Spring 3.0 will be the first release that requires Java 5+. From SpringSource's perspective this is nice because they can avail themselves to generics, varargs, etc. But from the app developer's perspective, the idea is that the platform is really pushing use of annotations as the preferred way of doing configuration. So that means Java 5+.

Spring 3.0 will continue to be compatible with J2EE 1.4 and JEE 5. Also, as JEE 6 matures, Spring 3.0 will try to provide early support, but progress on JEE 6 has been somewhat slow and Spring 3.0 should be out before JEE 6. So Spring 3.1 and 3.2 will probably continue work in this direction. Anyway, SpringSource plans to support JSF 2.0, JPA 2.0, Servlet 3.0 when it's ready, etc.)

<h3>Spring EL (Unified EL++)</h3>

Juergen described support for expression languages as probably the most important and powerful feature in Spring 3.0. It will be a proprietary EL parser implementation--one of the few new modules that Spring 3.0 will introduce--and it sounded like it would like Unified EL but on steroids. They're trying to keep the EL pretty similar to how it works in JSF, and Spring's EL is designed to be stylistically and syntactically compatible with Unified EL. Also, even though right now Spring Web Flow 2 uses external EL implementations (JBoss EL implementation of Unified EL, I think, or else OGNL), SWF 3 will use Spring EL instead of an external implementation.

Here's a code example he gave. This one is for EL in bean definitions:

<pre>&lt;bean class="mycompany.RewardsTestDatabase"&gt;
    &lt;property name="databaseName" value="#{systemProperties.databaseName}"/&gt;
    &lt;property name="keyGenerator" value="#{strategyBean.databaseKeyGenerator}"/&gt;
&lt;/bean&gt;</pre>

The idea is this. Just like in other ELs, you will have some implicit objects--objects that you can reference in your EL either because the objects are global in scope or else because the object lives in your current scope (e.g., request scope, session scope, etc.). So in the example above, <code>systemProperties</code> is a global implicit object, and <code>strategyBean</code> is a bean name. Bean names will be available as EL objects then.

I'm assuming that the <code>databaseName</code> and <code>keyGenerator</code> properties will be evaluated each time they are accessed. Otherwise it wouldn't be much of an expression language. I'm kind of curious to see how they do this. My guess is that the approach will be to proxy beans with EL-based properties, with the proxies evaluating the reference upon demand.

Juergen gave another example, this time of EL in component annotations. Here it is:

<pre>
@Repository
public class RewardsTestDatabase {

    @Value("#{systemProperties.databaseName}")
    public void setDatabaseName(String dbName) { ... }

    @Value("#{strategyBean.databaseKeyGenerator}")
    public void setKeyGenerator(KeyGenerator kg) { ... }
}
</pre>

I <i>believe</i> these two configuration examples are different in a subtle way. The value of the <code>@Value</code> attribute is a default value for the property if a value isn't explicitly defined in the app context. One practical benefit we get has to do with how component scanning currently works. When we component scan, we depend on autowiring to establish injected references. The problem is that if our dependencies are primitive types, autowiring is no longer an option and so we have to go explicit with our bean definitions. With EL-based defaults, we can keep the component scanning, and just set the primitive types using a bean, or maybe using system properties, etc.

<h3>REST support</h3>

Another big topic in Spring 3.0 is REST support. Spring Web MVC will provide first-class support for RESTful web services, and not necessarily just XML-based. Keith was saying that there will be a JSON view as well. So the web services would be useful not only in integration efforts but also for example when you have JavaScript widgets that want to make AJAX calls back to the server and get JSON. Anyway, Spring Web MVC's current design is such that it's really easy to tweak it a bit to support REST, and that's what they're doing. So one aspect of this would be to continue to use <code>@RequestMapping</code> to specify an HTTP method, such as <code>RequestMethod.PUT</code>.  Another piece would be to introduce a new <code>@PathVariable</code> annotation that works pretty much like <code>@RequestParam</code> currently works, but handles URL path segments instead of HTTP parameters. Here's a code example that shows how it will work:

<pre>
@RequestMapping(value = "/show/{id}", method = GET)
public Reward show(@PathVariable("id") long id) {
    return this.rewardsAdminService.findReward(id);
}
</pre>

I like it!

Another feature of the REST support is support for different representations, i.e. content negotiation. They want to use the HTTP <code>Accept</code> header as a new way to do view resolution. So presumably there will be a new <code>ViewResolver</code> implementation that looks at the <code>Accept</code> header and returns, say, an XML view (accepts <code>application/xml</code>), a JSON view (accepts <code>application/json</code>) or an ATOM view (accepts <code>application/atom+xml</code>) according to that header. This would be the preferred way of doing view resolution for the hardcore REST purists. But Spring Web MVC would also support extension-based resolution; e.g. <code>.json</code> maps to a <code>JsonView</code>, etc.

<h3>Some @MVC refinements</h3>

They're planning to add a couple of new @MVC annotations besides <code>@RequestParam</code> (which already exists) and the new <code>@PathVariable</code> annotation. These are:

<ul>
<li><code>@RequestHeader</code>: Grab HTTP request header data.</li>
<li><code>@CookieValue</code>: Grab cookie value. (Surprise!)</li>
</ul>

These will allow us to get at the data without having to declare the raw <code>HttpServletRequest</code> as we currently have to do.  Here's a code sample:

<pre>
@RequestMapping("/show")
public Reward show(@RequestHeader("region") long regionId,
    @CookieValue("language") String langId) {

    ...
}
</pre>

There's quite a bit more, but this post is pretty long as it is and I need to hit the sack. I'll try to post a little more tomorrow.

[Update: Here's <a href="http://springinpractice.wordpress.com/2008/12/03/new-stuff-in-spring-30-part-2/">part 2</a>.]

[Update: Here's <a href="http://www.infoq.com/news/2008/12/springone-2008">a related InfoQ article</a>.]
