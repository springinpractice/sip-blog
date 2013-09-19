---
layout: post
title: "Build a shopping cart with Spring Web Flow 2, part 1"
date: 2008-05-05 17:11:08
comments: true
categories: [Chapter 05 - Web Flow, Tutorials]
---
<div class="intro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/05/05/build-a-shopping-cart-with-spring-web-flow-2-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/05/06/build-a-shopping-cart-with-spring-web-flow-2-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/05/07/build-a-shopping-cart-with-spring-web-flow-2-part-3/">Part 3</a></span>
</div>

This is the first in a three-part series of posts showing how to get started with Spring Web Flow (SWF) 2.0 by building a simple shopping cart application. We'll do this as a series of steps:

<ol>
<li>First we'll just get Spring MVC working.</li>
<li>We'll use Spring Web Flow to create a minimal flow with a single state.</li>
<li>Finally we'll build the app out to contain multiple flows and subflows that make calls against backend logic.</li>
</ol>

SWF 2.0 is at the time of this writing very new, and there are several differences as compared to SWF 1.0.x. We will not however spend much time discussing those differences; instead we'll just focus on SWF 2.0.

To get the most out of this post, you should already be familiar with Spring in general and Spring MVC in particular.

<h3>An overview of Spring Web Flow</h3>

Spring Web Flow builds upon Spring MVC to support user-level, application-directed control flows.  For example, an e-commerce application may need to guide the end user through a checkout process that spans several web pages. The flow is not simply a multipage form (Spring MVC itself already supports that); rather it is a multistep <em>process</em> involving standard control flow constructions such as decisions ("please confirm or cancel your order"), loops ("we recommend products x, y, z... add as many as you like to your order") and subflows ("are you a new customer? please create an account"). In general, implementing such flows is not trivial.  SWF is an elegant solution to just this sort of problem.

To understand better, contrast application-directed interactions with the user-directed interactions that typically constitute the main part of an application's functionality. In user-directed interactions, the end user picks some function from a set of available functions, provides some parameters around that function, and then asks the application to carry it out.  For instance, the end user tells Photoshop to fill a selected region with <span style="background-color:#0F0">evil green</span>.  The app dutifully does just that, and control returns to the end user.

There are many cases, however, in which we want the application to drive a complex interaction between itself and the end user.  Perhaps the most obvious example is the one I mentioned above: the e-commerce checkout process.  See Figure 1 for one possible flow (and in fact this is the flow we're going to implement):

<div style="margin:20px 0;text-align:center">
<div><img src="http://wheelersoftware.s3.amazonaws.com/articles/spring-web-flow-2.0/flow.jpg" alt="Figure 1. A sample checkout flow." /></div>
<div class="caption">Figure 1. A sample checkout flow.</div>
</div>

In the checkout flow above, we have a starting state, an end state, and several intermediate states.  In the first state, we show the customer the contents of a shopping cart, along with some recommended products that the customer may want to buy.  The customer can add any of those products to the shopping cart, or he might decide to move forward by indicating whether he is a new or returning customer.  If he's a new customer, he'll need to create an account; otherwise he can just log in with his existing account.  In either case, he'll need to select a payment method, provide shipping information, and so forth.  At any step along the way he can cancel out of the checkout process.

<h4>Benefits of using Spring Web Flow</h4>

While you can certainly implement that flow without SWF, there are some challenges involved if you're doing it from scratch.  Let's take a look at some of those.

<strong>Understanding flow logic.</strong> For one, because the logic behind the flow itself is reasonably complex, it would be nice to be able to isolate the flow and its logic from the various pieces (like JSP pages, business logic, etc.) that make it up.  But the most straightforward implementation of flow logic would most likely involve distributing the flow logic (such as state transitions) across lots of different files.  So one challenge is that you either have to do a lot of extra work to externalize the flow logic, or else you have to live with that logic being distributed, which makes it much harder to understand.

<strong>Reusing flow logic.</strong> Another challenge is related to the fact that a flow has a logical structure that stands on its own, and in many cases you'd like to be able to reuse that&mdash;either across multiple apps, or else in multiple places within a single app.  This is hard to accomplish without an easy way to isolate that flow.

<strong>Getting the implementation right.</strong> A third challenge is just that it's easy to get the technical details wrong. For instance, what happens if the end user hits the browser's back button in the middle of the flow?  If the previous page had a form, we don't usually want the browser to confuse the user with a warning about resubmitting data or whatever.  Coding up flows from scratch means that you have to handle this sort of thing explicitly.  It would be nice not to have to mess around with this kind of thing&mdash;to have it handled automatically for you.

That's enough of an overview for us to get started.  Let's now look at setting up our sample project.

<h3>Spring MVC setup</h3>

Since Spring Web Flow is build on Spring MVC, let's start by getting Spring MVC working.  We'll create a simple home page for our shopping cart, and we'll serve this up using a plain old Spring controller (well, it will be annotated) rather than serving it using Spring Web Flow.  That's because the home page itself isn't part of any particular flow; it simply provides entry points into various flows, such as creating an account, logging in, adding an item to a shopping cart, and checking out.  Besides allowing us to make sure we have Spring MVC working before moving forward, this approach will also allow us to see how to integrate Spring MVC and Spring Web Flow.

For your convenience, here's a download of the minimalistic
(i.e. only Spring MVC, no SWF) shopping cart we're about to take a look at:

<center><span class="icon archive"><a href="http://wheelersoftware.s3.amazonaws.com/articles/spring-web-flow-2.0/mycart1.zip">mycart1.zip</a></span></center>

The download above does not include its dependencies.  You will need to grab those separately.  I've provided the links below.

<h4>Dependencies for mycart1.zip</h4>

These are all part of the Spring 2.5.4 distribution. Spring Web Flow 2.0 requires Spring 2.5.4 or higher. <a href="http://www.springframework.org/download">[download]</a>

<ul class="square">
<li><code>spring.jar</code> (located in <code>/spring-framework-2.5.4/dist</code>)</li>
<li><code>spring-webmvc.jar</code> (located in <code>/spring-framework-2.5.4/dist/modules</code>)</li>
<li><code>commons-logging.jar</code> (located in <code>/spring-framework-2.5.4/jakarta-commons</code>)</li>
</ul>

We will be adding dependencies as we progress; for the moment we're just getting Spring MVC set up.

<h4>Create a Spring MVC controller</h4>

Here's a very simple Spring MVC controller.  We'll be updating this over the course of the article.

<div>
<span class="code-listing">Code listing:</span> <code>mycart.CartController</code>
<pre>package mycart;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class CartController {

    @RequestMapping("/home.do")
    public void doHome() {
    }
}</pre>
</div>

This controller doesn't do much at all.  Basically we're using annotations to map <code>/home.do</code> requests to a JSP.

<h4>Create a JSP</h4>

Here's the home page JSP I just mentioned.  Like <code>CartController</code>, we'll be updating this.

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/jsp/home.jsp</code>
<pre>&lt;!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"&gt; 
&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;Products for Geeks - GeekWarez&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;h1&gt;Welcome to GeekWarez&lt;/h1&gt;
    &lt;/body&gt;
&lt;/html&gt;</pre>
</div>

<h4>Create web.xml</h4>

Here's our <code>web.xml</code> file:

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/web.xml</code>
<pre name="code" class="xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    version="2.5"&gt;
    
    &lt;!-- Spring MVC front controller. Automatically loads mycart-servlet.xml
         based on servlet name. --&gt;
    &lt;servlet&gt;
        &lt;servlet-name&gt;mycart&lt;/servlet-name&gt;
        &lt;servlet-class&gt;
            org.springframework.web.servlet.DispatcherServlet
        &lt;/servlet-class&gt;
    &lt;/servlet&gt;
    &lt;servlet-mapping&gt;
        &lt;servlet-name&gt;mycart&lt;/servlet-name&gt;
        &lt;url-pattern&gt;*.do&lt;/url-pattern&gt;
    &lt;/servlet-mapping&gt;
&lt;/web-app&gt;</pre>
</div>

All we're doing here is creating the Spring MVC <code>DispatcherServlet</code> front controller.  Because we've named it <code>mycart</code>, the default behavior for <code>DispatcherServlet</code> is to look for a Spring application context configuration file at <code>/WEB-INF/mycart-servlet.xml</code>, which we are about to see.

Eventually this front controller will handle not only our "normal" non-SWF requests, but also our SWF requests.  However I'm getting ahead of myself.

<h4>Create the Spring application context file</h4>

Here's <code>mycart-servlet.xml</code>, which <code>DispatcherServlet</code> loads as just explained:

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/mycart-servlet.xml</code>
<pre name="code" class="xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-2.5.xsd"&gt;
    
    &lt;!-- This activates post-processors for annotation-based config --&gt;
    &lt;!-- http://www.infoq.com/articles/spring-2.5-part-1 --&gt;
    &lt;context:annotation-config/&gt;
    
    &lt;context:component-scan base-package="mycart"/&gt;
    
    &lt;!-- Enables POJO @Controllers (like CartController) --&gt;
    &lt;bean class=
"org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping"/&gt;
    
    &lt;!-- Enables annotated methods on POJO @Controllers (like CartController) --&gt;
    &lt;bean class=
"org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/&gt;
    
    &lt;!-- Maps a logical view name to a physical resource --&gt;
    &lt;bean id="viewResolver" class=
"org.springframework.web.servlet.view.InternalResourceViewResolver"&gt;
        &lt;property name="prefix" value="/WEB-INF/jsp/"/&gt;
        &lt;property name="suffix" value=".jsp"/&gt;
    &lt;/bean&gt;
&lt;/beans&gt;</pre>
</div>

Nothing special here assuming you already know Spring MVC.

<h3>Milestone 1: Spring MVC is working</h3>

At this point you should be able to deploy the application.  Point your browser at

<center><code>http://localhost:8080/mycart1/home.do</code></center>

and you should get a very simple home page.  If so, congratulations, Spring MVC is working.

Now it's time to create our first flow using Spring Web Flow.

<div class="outro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/05/05/build-a-shopping-cart-with-spring-web-flow-2-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/05/06/build-a-shopping-cart-with-spring-web-flow-2-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/05/07/build-a-shopping-cart-with-spring-web-flow-2-part-3/">Part 3</a></span>
</div>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
