---
layout: post
title: "SpringOne 2GX takeaways"
date: 2010-10-28 12:06:28
comments: true
categories: [News]
---
<img src="http://springinpractice.s3.amazonaws.com/blog/images/springone2gx.png" alt="SpringOne 2GX logo" align="right" />

I attended SpringOne 2GX in Chicago last week and had a good time. Great sessions and keynotes, and I got to see some people I hadn't seen in a while, and meet some new people too.

Here are some of the things that I found most interesting. This is just my own personal list obviously, and there's a huge list of things happening in the Spring world.

**Code2Cloud.** This will be cloud-based application lifecycle management (ALM), like source control, issue tracking and continuous integration. One feature that looked particularly interesting was that the ALM system will execute the committed code and complain about code performance issues by creating an issue in the issue tracking system, complete with contextual information surrounding the performance issue in question.

**Gradle.** From what I gather, this is like Maven, but Groovy-based instead of XML-based, and much more aggressive about making assumptions about your project. The end result is that the build file is much smaller than a Maven POM file. I suspect that I'm not doing Gradle justice with this description but even with that description I'm interested because I like Maven's approach, but I wouldn't mind build files that are more compact.

**Spring support for NoSQL data stores.** NoSQL ("not only SQL") is a pretty important topic and Spring is jumping into this space with support for multiple providers. One thing that was neat was that you can have POJOs that are partially backed by traditional RDBMS databases (say for metadata) and partially backed by a NoSQL store, all using annotations as you might guess.

**Spring Social.** It allows developers to create apps that interact with the major social networks. Part of what's involved includes expanded <code>RestTemplate</code>s for making the web service calls. The other part is that it handles the OAuth authorization behind the scenes so developers don't have to mess around with what can be a complicated undertaking.

**Spring Mobile.** This allows Spring developers to develop apps for mobile platforms like the iPhone and the Droid.

**Support for HTML 5.** Saw some pretty nice demos of HTML 5 apps. I'm not sure at this point exactly what this support entails, but it looks like this is one of several areas of effort.

**Cache API and support for distributed caches.** Spring has included a preliminary cache API for several years now, but with the uptick in distributed caching they're building this out and will include it in Spring 3.1 I think they said.

**Support for environments.** Also a Spring 3.1 feature, there will be support for creating environment-specific beans (e.g. beans for dev, functional test, staging, prod, etc.).

**Gemfire data fabric.** I sat in on a session here and there was quite a bit to take in, so I'd need to hear the talk again to report on exactly what the offer is here. But the basic idea is highly available, low latency, horizontally scalable data that doesn't sacrifice consistency. I know a lot of organizations could use something like that so I'll be keeping an eye on this.

Those were the things that stood out for me. Any fears that Java and/or Spring aren't keeping with the times would appear to be unfounded.
