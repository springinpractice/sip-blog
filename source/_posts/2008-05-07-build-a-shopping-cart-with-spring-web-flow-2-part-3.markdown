---
layout: post
title: "Build a shopping cart with Spring Web Flow 2, part 3"
date: 2008-05-07 17:51:39
comments: true
categories: [Chapter 05 - Web Flow, Tutorials]
---
<div class="intro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/05/05/build-a-shopping-cart-with-spring-web-flow-2-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/05/06/build-a-shopping-cart-with-spring-web-flow-2-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/05/07/build-a-shopping-cart-with-spring-web-flow-2-part-3/">Part 3</a></span>
</div>

<h3>Building out the application</h3>

Now that we've gotten Spring MVC and SWF working, it's time to build out the shopping cart application.  Please download the following file before continuing:

<center><span class="icon archive"><a href="http://wheelersoftware.s3.amazonaws.com/articles/spring-web-flow-2.0/mycart3.zip">mycart3.zip</a></span></center>

We won't be looking at every aspect of this application, but I'll highlight the most interesting pieces from a SWF point of view.

<h4>Dependencies for mycart3.zip</h4>

Besides the dependencies we had for <code>mycart2.zip</code>, there are four others you'll need:

<ul class="square">
<li><code>cglib-nodep-2.1_3.jar</code></li>
<li><code>jstl.jar</code></li>
<li><code>standard.jar</code></li>
<li><code>sitemesh-2.3.jar</code> (from <a href="http://www.opensymphony.com/sitemesh/download.action">www.opensymphony.com/sitemesh/download.action</a>)</li>
</ul>

After you download <code>mycart3.zip</code> (and its dependencies), you can deploy it and then point your browser at

<p style="text-align:center">http://localhost:8080/mycart/home.do

Here are some notes to keep in mind:

<ul class="square">

<li>My registration and login flows don't actually "do" anything. They're not hooked up to Spring Security and they're not hooked up to a database.  So when you register you can just hit the submit button,and when you log in you can just hit the login button.  I just wanted to focus on the flows themselves.</li>

<li>I'm using Sitemesh to ensure a unified layout across the pages. Even if you are not familiar with Sitemesh, don't worry: it is very straightforward.  Basically you just put a servlet filter in front of the pages and the filter decorates the pages with a template (<code>/WEB-INF/jsp/pagetemplate.jsp</code> in this case) according to a simple configuration file (<code>/WEB-INF/decorators.xml</code>). If however you don't want to use Sitemesh, simply remove the filter from <code>web.xml</code> and Sitemesh is gone.</li>

</ul>

<h3>Multiple flows</h3>

Any given app may contain multiple flows, and <code>mycart3</code> is such an app.  We have four different flows:

<ul class="square">
<li><code>addToCart</code>: add an item to a shopping cart</li>
<li><code>checkout</code>: shopping cart checkout process</li>
<li><code>login</code>: log into the app</li>
<li><code>register</code>: register for a new user account</li>
</ul>

In some of the cases we have what we would intuitively consider a flow in that there are multiple states involved; in other cases there is only one state to speak of and so it may seem strange to regard the flow as being a flow.  However we'll see how that can make sense in the following section.

To create multiple flows, you will need to do the following. First, create the flow definition files and put them in your <code>/WEB-INF/flows</code> directory (or wherever you decided to put them).  Then add the flow locations to the flow registry, and the flow URL mappings to your <code>SimpleUrlHandlerMapping</code> (if that's what you're using), in your Spring application context, like so:

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/mycart-servlet.xml</code>
<pre name="code" class="xml">...

&lt;bean id="flowUrlMappings" class=
        "org.springframework.web.servlet.handler.SimpleUrlHandlerMapping"&gt;
    &lt;property name="mappings"&gt;
        &lt;value&gt;
            /addToCart.do=flowController
            /checkout.do=flowController
            /account/login.do=flowController
            /account/register.do=flowController
        &lt;/value&gt;
    &lt;/property&gt;
    &lt;property name="alwaysUseFullPath" value="true"/&gt;
&lt;/bean&gt;

&lt;flow:flow-registry id="flowRegistry" flow-builder-services="flowBuilderServices"&gt;
    &lt;flow:flow-location path="/WEB-INF/flows/addToCart.xml"/&gt;
    &lt;flow:flow-location path="/WEB-INF/flows/checkout.xml"/&gt;
    &lt;flow:flow-location path="/WEB-INF/flows/login.xml"/&gt;
    &lt;flow:flow-location path="/WEB-INF/flows/register.xml"/&gt;
&lt;/flow:flow-registry&gt;

...</pre>
</div>

Once you have those in place you should be set up for multiple flows.  Point your links at them and try them out!

<h3>Subflows</h3>

Going hand-in-hand with the idea of multiple flows is the idea that some flows might be subflows of other flows.  In <code>mycart3</code>,all four flows can be independently accessed, but in addition to that we have the <code>addToCart</code>, <code>login</code> and <code>register</code> flows being subflows to the <code>checkout</code> flow.  See Figure 3.

<div style="margin:20px 0;text-align:center">
    <div><img src="http://wheelersoftware.s3.amazonaws.com/articles/spring-web-flow-2.0/subflows.jpg" alt="Figure 3. Subflows." /></div>
    <div class="caption">Figure 3. Subflows.</div>
</div>

Here's the idea.  The three flows we've identified as subflows are defined as separate flows because clearly there are use cases where it makes sense for them to be accessed outside of a checkout process. For example, we want to be able to add products to a shopping cart while we're browsing the product catalog.  And of course we want people to be able to register and login even if they're not in a checkout process.

But those are also flows that we might want to include as part of a checkout process too.  During checkout, we might want to recommend products to the customer.  Or if the user hasn't yet logged in or registered, we'd want them to do that as part of the checkout process rather than forcing them to do that before they could enter the checkout process.

By defining the login, registration and add-to-cart as flows, we make them available both independently (as top-level flows) and also as part of a larger flow (like the checkout flow).  That's why we have these defined as flows even though they are currently implemented as single-state flows.  (It is however easy to imagine these being multi-state flows.  For example, the login flow may ask you to answer a challenge question if it detects that you're coming in from an unusual IP address, or the add-to-cart flow may ask you to enter a quantity before proceeding.)

So here is the checkout flow:

<div>
<span class="code-listing">Code listing:</span> <code>/WEB-INF/flows/checkout.xml</code>
<pre name="code" class="xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;flow xmlns="http://www.springframework.org/schema/webflow"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/webflow
        http://www.springframework.org/schema/webflow/spring-webflow-2.0.xsd"&gt;
    
    &lt;!-- Get the products one time, at the beginning of the flow --&gt;
    &lt;on-start&gt;
        &lt;set name="flowScope.products" value="cartService.products"/&gt;
        &lt;set name="flowScope.shippingOptions" value="cartService.shippingOptions"/&gt;
    &lt;/on-start&gt;
    
    &lt;!-- If not specified, the start state is the first state specified. --&gt;
    &lt;view-state id="viewCart" view="viewcart"&gt;
        &lt;!-- cart is available to SWF but not stored on the session under that
             name when using AOP proxy --&gt;
        &lt;on-render&gt;
            &lt;!-- Carry cart from Spring app context to request scope --&gt;
            &lt;set name="requestScope.shoppingCart" value="shoppingCart"/&gt;
            &lt;set name="requestScope.recommendations" value="cartService.recommendations"/&gt;
        &lt;/on-render&gt;
        &lt;transition on="addToCart" to="addProductToCart"/&gt;
        &lt;transition on="register" to="register"/&gt;
        &lt;transition on="login" to="login"/&gt;
    &lt;/view-state&gt;
    
    &lt;subflow-state id="addProductToCart" subflow="addToCart"&gt;
        &lt;!-- This is where we go when the subflow returns. productAdded is
             the name of an end-state. --&gt;
        &lt;transition on="productAdded" to="viewCart"/&gt;
    &lt;/subflow-state&gt;
    
    &lt;!-- New customers create a new account before moving forward --&gt;
    &lt;subflow-state id="register" subflow="register"&gt;
        &lt;transition on="accountAdded" to="paymentAndShipmentOptions"/&gt;
        &lt;transition on="cancelRegistration" to="viewCart"/&gt;
    &lt;/subflow-state&gt;

    &lt;!-- Existing customers log in before moving forward --&gt;
    &lt;subflow-state id="login" subflow="login"&gt;
        &lt!-- This is where we go when the subflow returns. productAdded is
             the name of an end-state. --&gt;
        &lt;transition on="loginOk" to="paymentAndShipmentOptions"/&gt;
    &lt;/subflow-state&gt;

    &lt;!-- Payment and shipment options --&gt;
    &lt;view-state id="paymentAndShipmentOptions" view="options"&gt;
        &lt;transition on="submit" to="confirmOrder"/&gt;
        &lt;transition on="back" to="viewCart"/&gt;
    &lt;/view-state&gt;
    
    &lt;!-- Confirm order --&gt;
    &lt;view-state id="confirmOrder" view="confirmorder"&gt;
        &lt;on-render&gt;
            &lt;set name="requestScope.shoppingCart" value="shoppingCart"/&gt;
        &lt;/on-render&gt;
        &lt;transition on="continue" to="thankYou"&gt;
            &lt;evaluate expression="cartService.submitOrderForPayment()"/&gt;
        &lt;/transition&gt;
    &lt;/view-state&gt;
    
    &lt;!-- Thank you page --&gt;
    &lt;view-state id="thankYou" view="thanks"&gt;
        &lt;transition on="continue" to="shop"/&gt;
    &lt;/view-state&gt;
    
    &lt;!-- Exit the flow, letting the user return to shopping --&gt;
    &lt;end-state id="shop" view="externalRedirect:contextRelative:/home.do"/&gt;
    
    &lt;global-transitions&gt;
        &lt;transition on="cancelCheckout" to="shop"/&gt;
    &lt;/global-transitions&gt;
&lt;/flow&gt;</pre>
</div>

In the code above, we are using the <code>&lt;subflow-state&gt;</code> element to call a subflow from a parent flow.  The <code>subflow</code> attribute specifies one of the flows you registered with the flow registry in the Spring configuration.  The flow starts at the subflow's start state, and continues until the subflow hits an end state.  The end state IDs provide keys that you can reference from the calling flow to effect state transitions.  For example, in the flow above,<code>accountAdded</code> is one of the end states for the <code>register</code> flows, and so one of the <code>&lt;transition&gt;</code> elements references that end state.

Recall from <code>register.xml</code> (see <a href="spring-web-flow-2.0-4.html">page 4</a>) that the end states specified a <code>view</code> attribute.  If the <code>register</code> flow is called directly, then SWF will use the <code>view</code> attribute to decide which view to show the user when the flow reaches a given end state.  If, however, the flow is called as part of a subflow (instead of being called directly), SWF will ignore the <code>view</code> attribute and instead follow whatever transition is defined in the calling <code>&lt;subflow-state&gt;</code>.

<h3>Using EL to call services</h3>

SWF allows you to make calls against service beans using an expression language (EL).  You can use either Unified EL or else OGNL for that, as mentioned earlier in the article.  We happen to be using OGNL though I understand that Unified EL and OGNL are mostly the same,at least where their core syntax is concerned.

There are various places in your flow where you might want to invoke service beans, and SWF provides various mechanisms for doing that.  Here's a table showing how to call services from different locations in your flow definition file:

<div class="table-with-caption">
    <div class="table">
        <table>
            <tr>
                <th>Location</th>
                <th>How to call the service bean</th>
            </tr>
            <tr>
                <td>On flow start</td>
                <td><code>/flow/on-start</code> element</td>
            </tr>
            <tr>
                <td>On state entry</td>
                <td><code>/flow/view-state/on-entry</code> element</td>
            </tr>
            <tr>
                <td>Immediately before rendering a view</td>
                <td><code>/flow/view-state/on-render</code> element</td>
            </tr>
            <tr>
                <td>On state exit</td>
                <td><code>/flow/view-state/on-exit</code> element</td>
            </tr>
            <tr>
                <td>On state transition</td>
                <td><code>/flow/view-state/transition/evaluate</code> element</td>
            </tr>
            <tr>
                <td>On flow end</td>
                <td><code>/flow/on-end</code> element</td>
            </tr>
        </table>
    </div>
    <div class="caption">Table 2. How to call service beans from different locations.</div>
</div>

Note that the above does not represent a 100% complete list of places where you can use EL, but it gives you most of the major cases and also the basic idea.

In <code>checkout.xml</code> we've used a number of the methods above.  For example, we use <code>&lt;on-start&gt;</code> to grab the product catalog and shipping options from the cart service and place them on the flow scope, which allows them to be used for the duration of the flow.  The <code>value</code> attribute on the <code>set</code> element is specified using EL.  (It looks a lot like JSP EL, if you are familiar with that.)  The <code>cartService</code> bean is available because we defined a Spring bean with that ID.

We also use <code>&lt;on-render&gt;</code> to prepare the <code>viewCart</code> state for rendering.  In this case, we place the user's shopping cart on the request scope so the JSP can easily reference it, and we also pull some recommendations off of the cart service and place those on the request scope as well.  Once again the <code>value</code> attribute is specified using EL.

<h3>Global transitions</h3>

At the end of <code>checkout.xml</code>, you will see that I've defined a <code>&lt;global-transitions&gt;</code> element.  This allows me to define a transition that applies to all states in the flow.  In this case, any time the user raises the <code>cancelCheckout</code> event, the global transition I've defined will kick in and carry the user to the <code>shop</code> state. Pretty handy for transitions that occur in multiple places throughout the flow.

<h3>That's our whirlwind tour</h3>

That concludes this tutorial on Spring Web Flow 2.0.  We've really only scratched the surface&mdash;for instance, we haven't even touched the new AJAX support that SWF 2.0 introduces&mdash;but this should give you an overall feel for how things work.  The SWF 2.0 distribution comes with a hotel booking sample application that shows you how to get SWF working with form validation and persistence as well (areas I've suppressed in this article so that I could focus on flow definition).

If you have suggestions and especially corrections to the article or the code, please leave them below.  SWF 2.0 is new so I have no doubt that you will be able to show me places where I could be doing something better.  Thanks!

<h3>Resources</h3>

A <a href="http://www.springhispano.org/?q=node/319">Maven version of the download</a> is available. The target link is in Spanish but there's a zip file at the bottom called <code>WebFlowExample.zip</code> that you can download. If you want to read the target link in English, you can use <a href="http://babelfish.yahoo.com/translate_txt">Alta Vista's free Babel Fish translation service</a>.

<div class="outro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/05/05/build-a-shopping-cart-with-spring-web-flow-2-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/05/06/build-a-shopping-cart-with-spring-web-flow-2-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/05/07/build-a-shopping-cart-with-spring-web-flow-2-part-3/">Part 3</a></span>
</div>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
