---
layout: post
title: "Acegi overview (now Spring Security)"
date: 2008-02-26 13:24:16
comments: true
categories: [Chapter 06 - Authentication, Chapter 07 - Authorization, Tutorials]
---
<div class="intro"><span class="icon stickyNote">I wrote this back when Spring Security was called Acegi. Now it's out of date, but I'm leaving it in the archive.</span></div>

Acegi has been around for a while, but I just recently tried it out and am impressed with it so far. Upon first glance Acegi is slightly intimidating because there are lots of classes involved, and it may be hard to keep them straight. Fortunately Acegi is actually straightforward, despite appearances, to anybody who already understands servlet filters and the basics of Spring bean configuration.

The overarching idea is that Acegi provides a suite of security services (mostly if not entirely around authentication and authorization) based on <a href="http://java.sun.com/products/servlet/Filters.html">servlet filters</a>. So for example there are filters for accepting user logins, enforcing access controls, catching security-related exceptions,handling logouts, handling "remember me" functionality for logins, switching from HTTP to HTTPS, etc. So using Acegi mostly involves deciding which of the filters you want (e.g., maybe you want "remember me", maybe you don't) and then configuring those into the app.

<h3>Acegi involves dependency injection into servlet filters (say what?)</h3>

Acegi does get a little interesting though because the various filters rely upon Spring's dependency injection, but the standard way of configuring servlet filters into <code>web.xml</code> does not support Spring dependency injection. (The <code>web.xml</code> schema doesn't have anything for dependency injection, nor should it.) Acegi uses a pretty neat trick to give us injectable servlet filters. What it does is it provides a <code>Filter</code> implementation called <code>FilterToBeanProxy</code> that you configure in like this:

<pre>&lt;filter&gt;
    &lt;filter-name&gt;security&lt;/filter-name&gt;
    &lt;filter-class&gt;org.acegisecurity.util.FilterToBeanProxy&lt;/filter-class&gt;
    &lt;init-param&gt;
        &lt;param-name&gt;targetClass&lt;/param-name&gt;
        &lt;param-value&gt;org.acegisecurity.util.FilterChainProxy&lt;/param-value&gt;
    &lt;/init-param&gt;
&lt;/filter&gt;
 
&lt;filter-mapping&gt;
    &lt;filter-name&gt;security&lt;/filter-name&gt;
    &lt;url-pattern&gt;/*&lt;/url-pattern&gt;
&lt;/filter-mapping&gt;</pre>

Pretty standard stuff. The idea behind <code>FilterToBeanProxy</code> is that it knows about your <code>[appname]-security.xml</code> bean configuration file, and so it can load in any beans that you define there. In <code>web.xml</code> we tell <code>FilterToBeanProxy</code> to load in a <code>FilterChainProxy</code> defined in <code>[appname]-security.xml</code>. <code>FilterChainProxy</code>,as its name suggests, is a <code>FilterChain</code> implementation. In turn, inside <code>[appname]-security.xml</code> we configure the various <code>Filter</code> beans mentioned above into the <code>FilterChainProxy</code>. Here's how it looks:

<pre>&lt;bean id="filterChainProxy" class="org.acegisecurity.util.FilterChainProxy"&gt;
    &lt;property name="filterInvocationDefinitionSource"&gt;
        &lt;!-- IMPORTANT: DON'T LINEBREAK THE FILTER LIST, OR ELSE BEAN LOOKUP BREAKS! --&gt;
        &lt;!-- I'M JUST DOING IT HERE FOR DISPLAY PURPOSES --&gt;
        &lt;value&gt;
            CONVERT_URL_TO_LOWERCASE_BEFORE_COMPARISON
            PATTERN_TYPE_APACHE_ANT
            /**=httpSessionContextIntegrationFilter,logoutFilter,
authenticationProcessingFilter,exceptionTranslationFilter,filterSecurityInterceptor
        &lt;/value&gt;
    &lt/property&gt;
&lt;/bean&gt;</pre>

The list of filter names is just a list of beans that you also configure in the same <code>[appname]-security.xml</code> config file. Here's an example:

<pre>&lt;bean id="httpSessionContextIntegrationFilter"
    class="org.acegisecurity.context.HttpSessionContextIntegrationFilter"/&gt;</pre>

Voil&agrave;, injectable servlet filters! Very cool.

<h3>Be careful of the following gotchas</h3>

There are some points/gotchas worth mentioning:

<ol>
<li>The servlet filters must be added in a specific order. If you get the order wrong, it won't work, since the servlet filters make assumptions about what kind of processing has already taken place. Check the <a href="http://www.acegisecurity.org/guide/springsecurity.html#filters">Acegi docs</a> for details.</li>
<li>If you're using other servlet filters (such as Sitemesh), the Acegi filters need to come before the others.</li>
<li>When configuring FilterChainProxy, you will need to map URL patterns to the list of filters that you want to service requests matching those patterns. For example, you might map <code>/**</code> (Ant syntax) to a list of four filters. The list is comma-delimited. It is important that you not include line breaks after the commas. I'm guessing you can't include whitespace at all, but for sure you can't include line breaks. Otherwise you will get <code>Bean not found: ''</code> exceptions, without any explanation as to what the actual problem is. I'm surprised they don't allow the line breaks since I figure many people will naturally want to break up a long list of filters with long names,but my surprise notwithstanding, the line breaks aren't allowed.</li>
</ol>

Here's a <a href="http://www.javaworld.com/javaworld/jw-10-2007/jw-10-acegi2.html">JavaWorld article on getting started with Acegi</a>. Also, <a href="http://www.amazon.com/Spring-Action-Craig-Walls/dp/1933988134/ref=pd_bbs_sr_1?ie=UTF8&s=books&qid=1199391787&sr=8-1">Spring in Action</a> has some good instructions.

<div class="endnote">Post migrated from my Wheeler Software site.</div>
