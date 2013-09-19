---
link: http://springinpractice.com/2008/12/01/rod-johnsons-keynote-address-at-springone-americas-2008/
layout: post
title: Rod Johnson's keynote address at SpringOne Americas 2008
date: 2008-12-01 22:44:13
comments: true
categories: [News]
---
I'm at SpringOne right now, and very much enjoying the <a href="http://www.diplomatresort.com/">Westin Diplomat</a>. My room is on the 21st floor and it overlooks the Atlantic. My balcony door is open and I can hear the waves crashing on the shore as I write. This is how life should always be. :-)

I attended Rod Johnson's keynote a couple of hours back. The keynote itself was informative, offering a pretty good glimpse into where SpringSource sees itself going over the coming year, and I had a chance to chat it up with Thomas Risberg and Colin Sampaleanu just a little bit since we were all sitting at the same table. Recognizing an opportunity when I see one, I asked them when they thought Spring 3 would be coming out and they laughed in a way that suggested that this wasn't the first time somebody had asked. Right now they're shooting for late March or thereabouts, though that was just their impression as opposed to a definite statement.

Anyway, here are some highlights from the keynote.

<h3>SpringSource in 2008</h3>

Rod started the keynote with some SpringSource efforts and accomplishments over 2008. I won't go into those in detail, but he mentioned several items, including
<ul>
	<li><a href="http://www.springsource.org/webflow">Spring Web Flow 2.0</a> GA: I remember it well as I wrote <a href="http://wheelersoftware.com/articles/spring-web-flow-2.0.html">an article</a> about it just before the GA and I wanted to make sure that nothing important had changed.</li>
	<li><a href="http://static.springsource.org/spring-batch/">Spring Batch</a> GA</li>
	<li>Launched new <a href="http://www.springsource.org/spring-integration">Spring Integration</a> project</li>
	<li>Preparing for Spring 3.0 GA</li>
	<li><a href="http://www.springsource.com/products/suite/dmserver">SpringSource dm Server</a></li>
	<li><a href="http://www.regdeveloper.co.uk/2008/01/29/springsource_buys_covalent/">Covalent Technologies acquisition</a>: ten years supporting Apache HTTP server and leading support provider for Tomcat (which is important to SpringSource's strategy, as I'll describe below)</li>
	<li><a href="http://broadcast.oreilly.com/2008/11/springsource-getting-into-the.html">G2One acquisition</a>: Bringing Groovy and Grails into SpringSource</li>
	<li>Several commercial software releases, including the <a href="http://www.springsource.com/products/suite/sts">SpringSource Tool Suite</a>, <a href="http://www.springsource.com/products/suite/ams">SpringSource Application Management Suite</a> and the <a href="http://www.springsource.com/products/suite/apfororacledb">SpringSource Advanced Pack for Oracle</a>.</li>
</ul>

They've clearly been a busy company over the past year, but it may not be entirely clear what they're up to. (Well, part of it is clear enough--they're moving into the app server space.) The next part of the keynote explained where SpringSource is going as a company.

<h3>SpringSource in 2009</h3>

Rod noted that 2008 has been a tough year for the world economy in general, and thus one reasonable prediction is that that's going to translate into pressure on IT budgets. One development he expects to see is license costs coming under increased scrutiny. Another is that he thinks IT will begin paying more attention to the costs associated with complexity, and will invest more in reducing that complexity.

I know from my own workplace that his ideas here are correct. Even before news broke that everything's melting down, we were reevaluating our license situation and indeed even in some cases making changes. And one of the themes that our CIO has sounded time and time again is that we need to work to reduce complexity when we can. I suspect that these are common themes anyway, but with the tough economic times it's pretty easy to believe that one can build a business strategy around this, and that's what SpringSource is doing.

<h3>The SpringSource strategy: help customers control costs by managing and reducing complexity</h3>

This was really the theme of the keynote and Rod noted that they'd even changed their tagline to reflect it: "Weapons for the War on Java Complexity." He explained how the various efforts over 2008 are generally aligned with managing complexity on different fronts. For example:

<b>Continue targeting complexity in development by expanding the Spring Framework in a way that simplifies application development.</b> Rod showed how with each new release of Spring, the SLOC for the pet store application dropped. He highlighted Spring Web MVC, Spring Web Flow and the acquisition of G2One in particular as representing a continued commitment to simplifying application development. Another notable item here includes Spring 2.5-style configuration via annotations.

Spring 3.0 moves the platform further along in this simplifying direction. Some highlights:
<ul>
	<li>Spring 3.0 will support EL in XML configuration and in annotations. He didn't say exactly where this would appear.</li>
	<li>Spring Web MVC will provide first-class support for RESTful web services. The match is strong (Spring Web MVC already has a @RequestMapping attribute that serves well here), and RESTful remoting is simpler than say SOAP-based web services.</li>
	<li>With Spring Web MVC, developers should adopt the annotated POJO model as Spring 3.0 will deprecate the Controller hierarchy and 3.1 will probably eliminate it altogether. Again the argument is that this is simpler than XML configuration.</li>
	<li>Spring Web Flow should be seen not as a niche technology but as a core part of their web framework that potentially applies any time there's a directed navigation requirement. Examples include wizards, back button, refresh, multiple UIs, and UI reuse.</li>
	<li>Speaking of their web framework, they're treating the following four projects as constituting a coherent, layered set: Spring Web MVC (the core), Spring JavaScript and Spring Web Flow (sitting on top of Spring Web MVC), and Spring Faces (sitting on top of the other three). He said that historically Spring hadn't simplified web development as much as it could have, and their converging the four aforementioned web technologies into a single group is an effort to address this.</li>
</ul>

Finally (we're still on the topic of managing complexity in development), Rod suggested that we take a look at <a href="http://static.springframework.org/spring/docs/2.5.x/api/org/springframework/beans/factory/annotation/Configurable.html">@Configurable</a> (which was available in Spring 2.0) and consider it a "secret weapon" against complexity.

The other major way in which SpringSource aims to manage/reduce complexity:

<b>Target complexity not only in development but also in operations:</b> Though it's by no means unanimous, lots of people feel that Spring has done a pretty nice job of simplifying JEE development. Their idea is now to take that to the data center. Their commercial offerings align with this; for example, the Application Management Suite provides deep diagnostics for Spring-based applications, and the dm Server aims to make release management simpler and more flexible. Besides their commercial offerings, their investment in Covalent Technologies (better access to Apache and Tomcat expertise) represents a desire to strengthen their offering in hosting and operations.

They see Tomcat as being the platform of choice where WAR-based deployments are concerned, but they see server-side OSGi as the wave of the future. Because Tomcat is currently widely deployed (I think Rod said something like 70% of enterprises have deployed Tomcat?), SpringSource wants to be involved there, and that's why they bought Covalent. But at the same time they are trying to encourage their customers and indeed the industry at large to make the transition to OSGi-based deployment.

And this leads to a couple of announcements he made.

<h3>A couple of product announcements</h3>

<b>tc Server:</b> The first announcement was that SpringSource will be releasing another server product to complement their current dm Server offering. This one is called tc Server for (you guessed it) Tomcat Server, and the idea is that it's an enhanced version of Tomcat, one that includes management functionality to make it "ready for the data center." The AMS product comes with it, and it will provide features such as visibility into Tomcat and app performance stats, deadlock detection, heap and thread dumps, and stuff like that. It will also be possible to manage tc Server WAR deployments through AMS. They're expecting (if I understood correctly) a GA release in January 2009.

Rod emphasized that the Spring Framework itself would remain server-agnostic, despite the fact that SpringSource is a server vendor.

<b>SpringSource Application Platform Configurator:</b> The second announcement is that SpringSource will be making available a web application called the SpringSource Application Platform Configurator. Again, managing complexity is the driver here. This time they are addressing the complexity associated with managing dependencies between JAR files, eliminating version conflicts, and that sort of thing. The idea is that you go to this app (which they host), and you specify a bunch of parameters such as the app type (web app, Grails, batch, etc.), whether you're OK with including pre-release software (such as milestone releases) or whether you allow only GAs, what sorts of capabilities you want and where your religious allegiances lie (is it OK to include JSF? is it OK to include Flex?), what hosting provider you're using (WebSphere, WebLogic, dm Server, etc.), what OS, etc. (By the way, they don't explicitly ask "where are your religious allegiances," but Rod noted that they recognize this reality of how software is sometimes chosen and the configurator is designed accordingly.) Then at the end you register (free), get a text dump of the JARs you just chose, and get a dynamically-generated ZIP containing those JARs (and they're of course mutually consistent so as to avoid versioning conflicts). That's a cool idea. They don't know though when this will be available for use; it's not even in beta yet.

Well, there you have it. If you weren't at the keynote, now it's as if you were.  :-)

I'll post more as the conference progresses. Here's <a href="http://greybeardedgeek.net/?p=66">another post</a> on the same keynote.

[Update: Here's <a href="http://www.infoq.com/news/2008/12/springone-2008">a related InfoQ article</a>.]
