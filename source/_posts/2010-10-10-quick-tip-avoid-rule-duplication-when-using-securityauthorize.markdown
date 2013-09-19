---
link: http://springinpractice.com/2010/10/10/quick-tip-avoid-rule-duplication-when-using-securityauthorize/
layout: post
title: Quick tip: avoid rule duplication when using security:authorize
date: 2010-10-10 22:56:42
comments: true
categories: [Chapter 07 - Authorization, Quick Tips]
---
Spring Security features a  tag that allows us to show or hide JSP content based on access rules we can define. Here's an example:

<pre style="margin:20px 0;">
&lt;security:authorize access="hasRole('admin')"&gt;
    &lt;a href="/main/admin.html"&gt;Admin&lt;/a&gt;
&lt;security:authorize&gt;
</pre>

This is probably the most common way to use the tag. The problem with this approach, though, is that it leads to rule duplication, because the same <code>hasRole('admin')</code> rule is defined in the security application context.

A better approach is to bind the display of the tag body to the user's actual access to the URL in question. Suppose for example that we have the following definition in the app context:

<pre style="margin:20px 0;">
&lt;intercept-url pattern="/main/admin.html" method="GET"
    access="hasRole('admin')" /&gt;
</pre>

Then we can simply replace the old JSP tag definition with a new one like this:

<pre style="margin:20px 0;">
&lt;security:authorize url="/main/admin.html" method="GET"&gt;
    &lt;a href="/main/admin.html"&gt;Admin&lt;/a&gt;
&lt;security:authorize&gt;
</pre>

Now we've successfully eliminated the duplicate rule definition. If we were to decide to change the rule to <code>hasRole('administrator')</code>, or to anything else for that matter, we'd be able to do that in a single location.
