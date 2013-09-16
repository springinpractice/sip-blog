---
layout: post
title: "Spring's constructor namespace is a bad idea"
date: 2012-05-07 21:26
comments: true
categories: [Chapter 01 - DI]
---
The other day I wrote up a post [explaining how to use Spring's constructor namespace](http://springinpractice.com/2012/04/26/the-spring-constructor-namespace-and-some-deep-thoughts/), which is new with Spring 3.1. So the following might be a little surprising:

*Spring's constructor namespace is a bad idea.*

I wanted to like it&mdash;honest I did. While I've always been a little iffy on the whole idea of introspecting on method parameter names, [Juergen Hoeller answered my concerns](https://issues.springsource.org/browse/SPR-6500?page=com.atlassian.jira.plugin.system.issuetabpanels:all-tabpanel) with respect to `@PathVariable` and the like. My worries around [least surprise](http://en.wikipedia.org/wiki/Principle_of_least_astonishment) were admittedly academic since I always compile binaries in debug mode.

So I tried the constructor namespace out, and it bit me in exactly the way I expected.

<!-- more -->

The problem
-----------

Just as a quick refresher (or introduction), here's how the constructor namespace works:

    <bean class="com.example.Client"
        c:restTemplate-ref-"restTemplate"
        c:baseUrl="http://localhost:8080/service" />

The configuration presupposes a constructor that looks like this:

    public Client(RestTemplate restTemplate, String baseUrl) { ... }

Here's the problem. Developers expect changing parameter names to be a local operation. Even in some of the Spring Web MVC cases that Juergen mentions in response to my JIRA issue&mdash;`@PathVariable`, `@RequestParam`, `@RequestHeader` and `@CookieValue`&mdash;the annotations in question are colocated with the parameters; e.g.

    public String getUser(@PathVariable Long id, Model model) { ... }

so somebody experienced with the framework will know what to do, and somebody less experienced will discover the issue soon enough (the code won't work) and fix it.

But the constructor namespace is totally different. Changing the constructor parameter name has the potential to break client code, and that client code may not be part of your current project. So you might not know you broke someone else's code.

This happened to me today, except it was breaking my own code in a separate project. I have a web service client with a constructor like this:

    public Client(RestTemplate restTemplate, String basePath) { ... }

and I changed that to

    public Client(RestTemplate restTemplate, String baseUrl) { ... }

This broke code in another project that was using the constructor namespace to inject values into the constructor. This is another violation of the principle of least surprise, but this time I don't think it's theoretical in the least. It happened to me the very first time.

Some possible solutions
-----------------------

One solution would be for Spring to simply get rid of this feature, as it leads to brittle code.

Another possibility would be for Spring to do something like this instead:

    public Client(@Param RestTemplate restTemplate, @Param String baseUrl) { ... }

and even

    public Client(@Param("restTemplate") RestTemplate template, @Param("baseUrl") String url) { ... }

That would make it more explicit that the constructor parameter names were being exposed as part of an API.

If you're set on using it, you should adopt the practice of documenting your constructors when they support this configuration style. And as an API consumer, you should assume that constructor parameter names are <em>not</em> contractual unless explicitly documented as such. Parameter names in Java have always counted as implementation details, and while I'm all for innovation and challenging the status quo, the benefit that the constructor namespace offers here is far too modest to call for revisiting this particular issue.
