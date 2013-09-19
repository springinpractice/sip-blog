---
link: http://springinpractice.com/2008/12/03/new-stuff-in-spring-30-part-2/
layout: post
title: New stuff in Spring 3.0, part 2
date: 2008-12-03 15:34:41
comments: true
categories: [News]
---
[Here's <a href="http://springinpractice.wordpress.com/2008/12/02/new-stuff-in-spring-30/">part 1</a> if you missed it.]

As promised, here's the rest of my notes dump on what's new in Spring 3.0, or really Spring 3.x since a  lot of what comes in Spring 3.x is waiting on JEE 6, which probably won't be finalized until after Spring 3.0 is already released.

<h3>More about Spring 3 and its support for JEE 6</h3>

Before I continue describing Juergen's first talk, I did attend another of his talks just a while ago and it seems like Spring 3/JEE 6 integration is going to be "interesting," especially as concerns Spring integration with the forthcoming Java Web Beans effort (and to a lesser extent, EJB 3.1).  He didn't work too hard to hide his displeasure at the unattributed fashion in which many of the core ideas from Spring are making their way into the JEE specs. He described Web Beans as being a standardized DI framework with a focus on supporting scoped beans (request, conversation, session, application, etc.), and noted that many of the annotations from Web Beans and EJB 3.1 have direct counterparts in Spring. Taken together it sounds like Web Beans + EJB 3.1 will compete with Spring, with Web Beans handling the user component model and EJB 3.1 providing platform services (transaction management, container-managed security and so forth).

Anyway regarding Spring 3 and JEE 6, it will be fun to watch as it unfolds.  :-)

Here's some more information related to how Spring 3 will support JEE 6:

<ul>
<li>JSF 2.0: full compatibility as managed bean facility</li>
<li>JPA 2.0: support for lock modes and query timeouts (Hibernate supports these, but this is new for JPA)</li>
<li>JAX-RS and Jersey reference implementation (part of Glassfish): Spring will support these though they see JAX-RS as an alternative to Spring Web MVC's REST support. Juergen said that he sees JAX-RS for more "hardcore" RESTful web service applications, and it's a completely separate component model for building programmatic resource endpoints.</li>
<li>JSR 236 WorkManager and TimerManager: Rich, standardized access to thread pools. This spec is behind schedule so it won't be in Spring 3. Maybe Spring 3.1. Juergen is excited about this though it sounded like it's more from the framework perspective than from the app developer perspective.</li>
<li>Servlet 3.0: This won't be supported til Spring 3.1 or Spring 3.2, but two major items here are (1) auto-deployment of framework integration classes (I didn't fully catch what this meant--I couldn't tell if it's about hot-deploying JARs or what), and (2) support for Comet requests. The Comet support would include new HTTP request and response types.
<li>EJB 3.1 and Web Beans as described above. Just depends where the specs go.</li>
</ul>

The Spring team is tracking GlassFish 3 and Tomcat 7 as well.

<h3>JSR 303: Bean Validation</h3>

This one looks like it will be for Spring 3.1 rather than Spring 3.0 since the Bean Validation spec isn't done yet. The idea though is that JSR 303 provides a declarative, annotation-based model for defining validation constraints, and so instead of forcing developers to invoke validation manually, the framework would automatically invoke it at appropriate places.  I haven't played much with Hibernate Validator yet but I think you can make Hibernate automatically validate persistent objects before saving them.  So the same sort of thing would occur elsewhere.  For example Spring Web MVC will I believe automatically invoke validation after doing form binding.

I did ask about the Bean Validation Framework (from the Spring Modules project), and whether that was dead. I think for a brief moment he didn't know what I was talking about when I said "Bean Validation Framework" (not a good sign for the BVF), and when I mentioned Spring Modules he said [paraphrasing here]: yeah, BVF is pretty much dead, use Hibernate Validator instead. My brother asked Uri Boness (BVF lead) about it several months back and he said roughly the same thing, that the best bet is for people to use Hibernate Validator and plan on JSR 303 being the future of validation.

Spring 3.0 will support Hibernate Validator annotations but the real goal is to support JSR 303 annotations.

<h3>Portlet 2.0 support</h3>

Some major Portlet 2.0 capabilities include:

<ul>
<li>An explicit action name concept for dispatching</li>
<li>Two new request types: resource requests for servlet-style serving, and events for inter-portlet communication</li>
<li>Portlet filters, which are analogous to servlet filters</li>
</ul>

Spring 3 will have Portlet 2.0 support. In particular, Portlet MVC 3.0 will support the following mapping annotations: <code>@ActionMapping</code>, <code>@RenderMapping</code>, <code>@ResourceMapping</code>, and <code>@EventMapping</code>. Conceptually these are specializations of Spring Web MVC's <code>@RequestMapping</code> annotation even though there's no explicit subtype relationship here.

Some other odds and ends:

<ul>
<li>In the Spring EL there will be some scope-sensitive Portlet-related implicit objects such as the current <code>PortletRequest</code> and <code>PortletSession</code>.</li>
<li>The <code>@RequestHeader</code> and <code>@CookieValue</code> annotations I mentioned yesterday will work for both Servlet MVC and Portlet MVC.</li>
</ul>

Juergen also described some areas of research. These aren't necessarily going into Spring 3 anytime soon but they're just taking a look right now.

<h3>Research area #1: Conversation management</h3>

A key problem in conversation management is isolating concurrent windows in the same browser. Historically this has been the domain of Spring Web Flow, but there's some thought of moving some level of conversation management to Spring Core. The idea is to introduce a new scope that's larger than a single request but smaller than an entire session--a conversation--and be able to handle that, along the lines of the MyFaces Orchestra project.

This is just me editorializing now, but I'm myself a little dubious of the usefulness of a conversation scope if the goal is really just managing different browser windows. To me it seems like running a single app in multiple browser windows is a corner case. I suspect that what's going on is that Spring is responding to the fact that JEE 6 Web Beans will have a conversation scope.

If the goal is to handle Spring Web Flow-type scenarios (single window) then yeah, that I can see. Anyway.

<h3>Research area #2: Scoped bean serializability</h3>

Sometimes you want to be able to serialize beans, but you can't, because they contain references (either directly or indirectly) to unserializable objects, like e.g. DataSources. They're looking at ways to address this.

I'm not sure what sorts of serialization scenarios they have in mind. Originally I was thinking Juergen was talking about saving out sessions during server shutdowns (like Tomcat does) and stuff like that, but as I write this, that doesn't seem right, because your domain objects wouldn't have DataSource references. So apparently some people want to save managed objects, which seems odd since those managed objects (controllers, service beans, DAOs, etc.) are usually stateless. Oh well. I don't doubt that somebody has some reasonable use case here; I just don't myself know exactly what people have in mind on this one.

Hm... but scoped beans would typically be domain objects... heh, I'm totally confused on this one.  :-)

Anyway, the Spring team is thinking about how best to handle this. Juergen offered that one "solution" is just to design your apps so that they don't involve serializing scoped beans. Another idea might be to have serializable proxies that are able to reconstitute the scoped bean when the proxy is initialized.

Spring 3 will also address a handful of housekeeping items.

<h3>Housekeeping and portfolio rearrangements</h3>

Some portfolio rearrangements include:

<ul>
<li>There will be a revised OXM modules as part of the core, rather than just being part of Spring Web Services, as there are broader areas of applicability, including XML payloads in REST and XML datatypes in the database.</li>
<li>The binding and type conversion infrastructure will be revised. It will include capabilities from Spring Web Flow's binding, and there will be EL integration here.</li>
<li>Spring 3.0 will include the core functionality of Spring JavaConfig, which basically allows you to configure application contexts from Java instead of using XML.</li>
</ul>

<h3>Deprecation and pruning</h3>

The following will be deprecated:

<ul>
<li>The traditional MVC controller class hierarchy (deprecated in Spring 3.0, removed entirely in Spring 3.1).</li>
<li>Traditional JUnit 3.8 test class hierarchy (this will be replaced with the test context framework)</li>
<li>Various outdated helper classes</li>
</ul>

The following will be removed in Spring 3.0:

<ul>
<li>Commons Attributes support</li>
<li>TopLink API support (replace with JPA/EclipseLink)</li>
<li>Support for Struts 1.x subclass-style actions</li>
</ul>

<strong>Some related links:</strong>

<ul>
<li><a href="http://greybeardedgeek.net/?p=83">SpringOne Americas 2008 - Day 3</a></li>
<li><a href="http://ptrthomas.wordpress.com/2008/12/04/springone-2008-day-3/">SpringOne 2008 Day 3</a></li>
</ul>

[Update: Here's <a href="http://www.infoq.com/news/2008/12/springone-2008">a related InfoQ article</a>.]
