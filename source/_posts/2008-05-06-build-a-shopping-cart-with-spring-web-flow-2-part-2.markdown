---
link: http://springinpractice.com/2008/05/06/build-a-shopping-cart-with-spring-web-flow-2-part-2/
layout: post
title: Build a shopping cart with Spring Web Flow 2, part 2
date: 2008-05-06 17:50:53
comments: true
categories: [Chapter 05 - Web Flow, Tutorials]
---
<div class="intro">
<span class="icon stickyNote">This post is part of a three part series: <a href="http://springinpractice.com/2008/05/05/build-a-shopping-cart-with-spring-web-flow-2-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/05/06/build-a-shopping-cart-with-spring-web-flow-2-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/05/07/build-a-shopping-cart-with-spring-web-flow-2-part-3/">Part 3</a></span>
</div>

<h3>Creating a simple flow: Spring application context configuration</h3>

Now we have Spring MVC working, so it's time to expand on that and get Spring Web Flow working too.  We'll start by creating a very simple flow&mdash;one that has only a single view state.  First we'll look at the updated Spring application context configuration.  After that we'll look at the flow definition itself.

To make it easier for you to follow along, here's the updated,one-state version of our shopping cart:

<center><span class="icon archive"><a href="http://wheelersoftware.s3.amazonaws.com/articles/spring-web-flow-2.0/mycart2.zip">mycart2.zip</a></span></center>

<h4>Dependencies for mycart2.zip</h4>

Because we're now using SWF, we have additional dependencies. Here's the full set so far:

These come from the Spring 2.5.4 distribution. <a href="http://www.springframework.org/download">[download]</a>
<ul class="square">
<li><code>spring.jar</code> (located in <code>/spring-framework-2.5.4/dist</code>)</li>
<li><code>spring-webmvc.jar</code> (located in <code>/spring-framework-2.5.4/dist/modules</code>)</li>
<li><code>commons-logging.jar</code> (located in <code>/spring-framework-2.5.4/jakarta-commons</code>)</li>
</ul>

You will need the following JARs from the Spring Web Flow 2.0 distribution. <a href="http://www.springframework.org/download">[download]</a>

<ul class="square">
<li><code>spring-webflow-2.0.0.jar</code></li>
<li><code>spring-binding-2.0.0.jar</code></li>
<li><code>spring-js-2.0.0.jar</code></li>
</ul>

You will need the following JAR from the <a href="http://www.ognl.org/">OGNL web site</a>:

<ul class="square">
<li><code>ognl-2.6.9.jar</code></li>
</ul>

Spring Web Flow uses either OGNL or Unified EL for parsing expressions.  For this article I arbitrarily picked OGNL though you can use <code>jboss-el.jar</code> (currently the only Unified EL implementation that will work with SWF) too.

<h4>web.xml and mycart.CartController</h4>

For now we aren't making any changes to either of these at all.

<div style="margin:10px 0 20px 20px;float:right">
<div><img src="http://wheelersoftware.s3.amazonaws.com/articles/spring-web-flow-2.0/swf-config.gif" alt="Figure 2. Overview of SWF components." /></div>
<div class="caption">Figure 2. Overview of SWF components.</div>
</div>

<h4>Spring application context overview</h4>

Before jumping into the code, let's do a quick overview of the various components involved.  Please see Figure 2.

We already saw <code>DispatcherServlet</code> in <code>web.xml</code>.  (So it's not part of <code>mycart-servlet.xml</code>; we're discussing it here just to show the relationship between the servlet and the <code>FlowController</code>.)  <code>DispatcherServlet</code> is the Spring MVC front controller and it receives any Spring MVC requests&mdash;including SWF requests, as SWF is based on Spring MVC&mdash;according to your <code>web.xml</code> configuration.

We're using <code>SimpleUrlHandlerMapping</code> to map flow requests from <code>DispatcherServlet</code> to <code>FlowController</code>.

So now <code>FlowController</code> has the request. <code>FlowController</code> is just a Spring MVC controller that receives flow requests and passes them to <code>FlowExecutor</code> for actual processing.

<code>FlowExecutor</code> contains the actual logic for processing Spring Web Flow requests.  Among other things it determines for any given request which flow is involved and figures out what state transition to enact, based on the request.

When <code>FlowExecutor</code> needs a flow, it grabs the flow from the <code>FlowRegistry</code>, which is responsible for loading the flow from a flow configuration file and maintaining it on behalf of the <code>FlowExecutor</code>.

<code>FlowBuilderServices</code> is just a container for various services that <code>FlowRegistry</code> needs when constructing flows. This includes, for example, a service to provide for the creation of view factories.  In our example, we will see that we're specifically using a <code>MvcViewFactoryCreator</code>, which creates view factories for Spring MVC views.

Finally, we define a <code>ViewResolver</code>, which is a Spring MVC interface that carries logical view names to physical resources (such as JSPs).  For example, a <code>ViewResolver</code> might carry the logical name <code>checkout/viewcart</code> to the physical resource <code>/WEB-INF/jsp/checkout/viewcart.jsp</code>.

<div style="clear:both"></div>

<h4>Spring application context configuration file</h4>

And finally here's the configuration file itself:

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/mycart-servlet.xml</code>
<pre name="code" class="xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:flow="http://www.springframework.org/schema/webflow-config"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-2.5.xsd
    http://www.springframework.org/schema/webflow-config
    http://www.springframework.org/schema/webflow-config/spring-webflow-config-2.0.xsd"&gt;


    &lt;!-- SPRING MVC STUFF --&gt;

    &lt;!-- This activates post-processors for annotation-based config --&gt;
    &lt;!-- http://www.infoq.com/articles/spring-2.5-part-1 --&gt;
    &lt;context:annotation-config/&gt;

    &lt;context:component-scan base-package="mycart"/&gt;

    &lt;!-- Enables POJO @Controllers (like CartController) --&gt;
    &lt;bean class=
"org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping"/&gt;
    
    &lt;!-- Maps flow requests from DispatcherServlet to flowController --&gt;
    &lt;bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping"&gt;
        &lt;property name="mappings"&gt;
            &lt;value&gt;
                /account/register.do=flowController
            &lt;/value&gt;
        &lt;/property&gt;
        &lt;property name="alwaysUseFullPath" value="true"/&gt;
    &lt;/bean&gt;
    
    &lt;!-- Enables annotated methods on POJO @Controllers (like CartController) --&gt;
    &lt;bean class=
"org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/&gt;
    
    &lt;!-- Enables plain Controllers (e.g. FlowController) --&gt;
    &lt;bean class=
"org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/&gt;
    
    &lt;!-- Maps a logical view name to a physical resource --&gt;
    &lt;bean id="viewResolver" class=
"org.springframework.web.servlet.view.InternalResourceViewResolver"&gt;
        &lt;property name="prefix" value="/WEB-INF/jsp/"/&gt;
        &lt;property name="suffix" value=".jsp"/&gt;
    &lt;/bean&gt;
    
    
    &lt;!-- SPRING WEB FLOW STUFF --&gt;
    
    &lt;bean id="flowController" class=
"org.springframework.webflow.mvc.servlet.FlowController"&gt;
        &lt;property name="flowExecutor" ref="flowExecutor"/&gt;
    &lt;/bean&gt;
    
    &lt;flow:flow-executor id="flowExecutor" flow-registry="flowRegistry"/&gt;
    
    &lt;!-- This creates an XmlFlowRegistryFactory bean --&gt;
    &lt;flow:flow-registry id="flowRegistry"
            flow-builder-services="flowBuilderServices"&gt;
        &lt;flow:flow-location path="/WEB-INF/flows/register.xml"/&gt;
    &lt;/flow:flow-registry&gt;
    
    &lt;flow:flow-builder-services id="flowBuilderServices"
            view-factory-creator="viewFactoryCreator"/&gt;
    
    &lt;bean id="viewFactoryCreator" class=
"org.springframework.webflow.mvc.builder.MvcViewFactoryCreator"&gt;
        &lt;property name="viewResolvers"&gt;
            &lt;list&gt;
                &lt;ref bean="viewResolver"/&gt;
            &lt;/list&gt;
        &lt;/property&gt;
    &lt;/bean&gt;
&lt;/beans&gt;</pre>
</div>

Now let's look at our basic flow definition, which will initially at least be much simpler than the application context file.

<h3>Creating a Simple Flow: Flow Definition File and JSPs</h4>

We'll now add a SWF flow definition file, update our original home page JSP, and add a new JSP for registering as a new customer.

<h4>The flow definition file</h4>

Here's a definition for our first flow, which will be a bare-bones registration process.  We'll add more flows to the application but this is our starting point just to get SWF working.  You'll need to create a directory <code>/WEB-INF/flows</code>, and then add the following flow definition file to that directory, naming the file <code>register.xml</code>.

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/flows/register.xml</code>
<pre name="code" class="xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow
    http://www.springframework.org/schema/webflow/spring-webflow-2.0.xsd"&gt;
    
    &lt;!-- By default, the first state is the start state. --&gt;
    &lt;view-state id="register" view="account/registerForm"&gt;
        &lt;transition on="submitRegistration" to="accountAdded"/&gt;
        &lt;transition on="cancelRegistration" to="cancelRegistration"/&gt;
    &lt;/view-state&gt;
    
    &lt;end-state id="accountAdded"
        view="externalRedirect:contextRelative:/home.do"/&gt;
    &lt;end-state id="cancelRegistration"
        view="externalRedirect:contextRelative:/home.do"/&gt;
&lt;/flow&gt;</pre>
</div>

Here we have a single state, called a <em>view state</em>, and all it does is show us the hardcoded JSP that we created earlier.  We're identifying the state itself as <code>register</code>, and the logical view name associated with this view state is <code>account/registerForm</code>.  This is the name that the view resolver we defined in the Spring app context file will map to a physical location; in this case the physical location will be <code>/WEB-INF/jsp/account/registerForm.jsp</code>.

By default, SWF interprets the first state in the file as being the <em>start state</em>, or entry point into the flow.  There must be exactly one start state.  The flow itself is called <code>register</code> (this is because we've named the flow definition file <code>register.xml</code>).  As it happens we've also named the first state <code>register</code> though there's no requirement for the start state to have the same name as the flow.

I've defined a couple of transitions out of the <code>register</code> state.  One transition, <code>submitRegistration</code>, responds to registration submission events on the user interface, such as the user clicking a submit button on a registration form.  The other transition, <code>cancelRegistration</code>, responds to cancelation events on the UI, such as the user clicking a cancel link.  Each of these transitions leads to another state in the flow.  In this case, both of the transitions lead to <em>end states</em>, which are exits out of the flow, but transitions can lead to other view states (or even other sorts of state) as well.  As shown there can be multiple end states for a flow.  Here my end states happen to be doing the same thing; they're redirecting the user to a context-relative (as in relative to the servlet context) path <code>/home.do</code>, which you will recall is just the home page.  There are some other relativizations you can do as well:

<div class="table-with-caption">
<div class="table">
<table>
<tr>
<th>Relativization</th>
<th>Example</th>
</tr>
<tr>
<td>Servlet mapping-relative</td>
<td><code>externalRedirect:/hotels/index</code></td>
</tr>
<tr>
<td>Servlet-relative path</td>
<td><code>externalRedirect:servletRelative:/hotels/index</code></td>
</tr>
<tr>
<td>Context-relative path</td>
<td><code>externalRedirect:contextRelative:/dispatcher/hotels/index</code></td>
</tr>
<tr>
<td>Server-relative path</td>
<td><code>externalRedirect:serverRelative:/springtravel/dispatcher/hotels/index</code></td>
</tr>
<tr>
<td>Absolute URL</td>
<td><code>externalRedirect:http://www.paypal.com/</code></td>
</tr>
</table>
</div>
<div class="caption">Table 1. Relativizations for externalRedirect.</div>
</div>

Anyway we're doing the context-relative redirection in this case. This is how we jump out of SWF and return control to the application. (Or how we return control to a calling flow, but we'll get to that.)

<h4>The JSPs</h4>

Now let's revisit our home page JSP.  It's still pretty similar but I'm adding a registration link.

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/jsp/home.jsp</code>
<pre name="code" class="html">&lt;!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"&gt; 
&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;Products for Geeks - GeekWarez&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;h1&gt;Welcome to GeekWarez&lt;/h1&gt;
        
        &lt;div&gt;&lt;a href="account/register.do"&gt;Register&lt;/a&gt;&lt;/div&gt;
    &lt;/body&gt;
&lt;/html&gt;</pre>
</div>

The registration link takes us into the <code>register</code> flow,since the last path segment is <code>register.do</code>.  In our Spring app context we told the flow registry about <code>register.xml</code>, so SWF will know to map requests for <code>/account/register.do</code> to the <code>register</code> flow.

Now we need a registration form.  The following page is our current version of a registration "form", but as you can see it isn't really a form at all (yet).  It is just a couple of links:

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/jsp/account/registerForm.jsp</code>
<pre name="code" class="html">&lt;!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"&gt; 
&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;Register - GeekWarez&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;h1&gt;Register&lt;/h1&gt;
        
        &lt;div&gt;
            &lt;a href="${flowExecutionUrl}&amp;_eventId=submitRegistration"&gt;Submit&lt;/a&gt; |
            &lt;a href="${flowExecutionUrl}&amp;_eventId=cancelRegistration"&gt;Cancel&lt;/a&gt;
        &lt;/div&gt;
    &lt;/body&gt;
&lt;/html&gt;</pre>
</div>

The point of the two links is to give the user a way to generate <code>submitRegistration</code> and <code>cancelRegistration</code> events.  We saw in the flow definition that these two events trigger state transitions.  Of special importance is the URL for each of these links.  Notice that by using <code>${flowExecutionUrl}</code>, we're still pointing the user at the same <code>/account/register.do</code> servlet path. That's because we're still working within the <code>register</code> flow.  The <code>${flowExecutionUrl}</code> URL includes an <code>execution</code> HTTP parameter whose value is a key that SWF uses for flow-tracking purposes.  In addition we add an <code>_eventId</code> parameter that we use to drive state transitions.  The event IDs are what we're referencing when we define transitions in the flow definition file.

So really so far all we can do at this point is move back and forth between the home page and the registration page, but we're doing it with Spring Web Flow.

<h3>Milestone 2: Spring Web Flow is working</h3>

Point your web browser at

<center><code>http://localhost:8080/mycart2/home.do</code></center>

making any adjustments you need to make for the port or application context.  Also note that the context is now <code>mycart2</code> instead of <code>mycart1</code> like it was in the first version of the code.  If you're able to click back and forth between the home page and the registration page, then congrats, Spring Web Flow 2.0 is working!  Celebrate!

<div class="outro">
<span class="icon stickyNote">This post is part of a three part series: <a href="http://springinpractice.com/2008/05/05/build-a-shopping-cart-with-spring-web-flow-2-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/05/06/build-a-shopping-cart-with-spring-web-flow-2-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/05/07/build-a-shopping-cart-with-spring-web-flow-2-part-3/">Part 3</a></span>
</div>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
