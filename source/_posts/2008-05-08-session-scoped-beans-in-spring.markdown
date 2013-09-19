---
link: http://springinpractice.com/2008/05/08/session-scoped-beans-in-spring/
layout: post
title: Session-scoped beans in Spring
date: 2008-05-08 00:50:20
comments: true
categories: [Chapter 01 - DI, Tutorials]
---
This tutorial will show you how to define session-scoped beans in Spring.  The idea here is that you have a web application, and you have various objects that exist on a per-session basis, such as maybe a user profile or a shopping cart. You'd like to make those available to some service bean, say, without having to manually pull them off the session and pass them as method arguments every time.  First I'll show you how to do that.  After that I'll talk a little bit about whether I think it's a good idea.

Here's the code for this article: <a href="http://wheelersoftware.s3.amazonaws.com/articles/spring-session-scoped-beans/ssb-example.zip">ssb-example.zip</a>

The code uses annotation-based autowiring. If you are unfamiliar with that and it's not reasonably clear what I'm doing with the code, you might want to see my articles <a href="#">Annotation-Based Autowiring in Spring 2.5</a> and <a href="#">Annotation-Based MVC in Spring 2.5</a>.

<h3>Dependencies</h3>

I'm using Java 6 (though Java 5 should work too) and Spring 2.5.4 (though Spring 2.5.x should work generally).  You will need the following JARs:

<ul>
    <li><code>cglib-nodep-2.1_3.jar</code></li>
    <li><code>commons-logging.jar</code></li>
    <li><code>spring.jar</code></li>
    <li><code>spring-webmvc.jar</code></li>
</ul>

<h3>Java classes</h3>

We have four classes: a controller, a service interface, a service implementation, and a model class.

First, the controller:

<pre>package ssbexample;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class MyController {
    private String SHOPPING_CART_KEY = "shoppingCart";
    
    private MyService service;

    public MyService getService() {
        return service;
    }

    public void setService(MyService service) {
        this.service = service;
    }
    
    @RequestMapping("/viewcart.do")
    public ModelMap viewCart() {
        return new ModelMap(SHOPPING_CART_KEY, service.getShoppingCart());
    }
    
    @RequestMapping("/additem.do")
    public ModelAndView addItem() {
        service.addItem();
        ShoppingCart cart = service.getShoppingCart();
        return new ModelAndView("/viewcart", SHOPPING_CART_KEY, cart);
    }
}</pre>

The thing to notice about the controller code is that I'm grabbing the shopping cart from the service bean itself, rather than pulling it off of the session directly.  That means that the service bean is somehow able to return a session-specific shopping cart.  (And we'll see how to do that shortly.)  We're also able to tell the service to add an item (to the cart), and the service once again knows exactly which cart to add the item to.

Now here's the service interface:

<pre>package ssbexample;

public interface MyService {
    
    ShoppingCart getShoppingCart();
    
    void addItem();
}</pre>

Here's the service implementation:

<pre>package ssbexample;

import org.springframework.stereotype.Service;

@Service("service")
public class MyServiceImpl implements MyService {
    private ShoppingCart shoppingCart;

    public ShoppingCart getShoppingCart() {
        return shoppingCart;
    }
    
    public void setShoppingCart(ShoppingCart shoppingCart) {
        this.shoppingCart = shoppingCart;
    }
    
    public void addItem() {
        shoppingCart.addItem();
    }
}</pre>

Notice that with the shopping cart getter and setter, we have a way to inject a shopping cart into the service bean.  But we have only a single service bean instance, so it looks a little bit like magic right now.

And finally, here's the model class itself (a simple shopping cart):

<pre>package ssbexample;

import java.io.Serializable;

public class ShoppingCart implements Serializable {
    private int numItems;
    
    public ShoppingCart() {
    }
    
    public int getNumItems() {
        return numItems;
    }
    
    public void addItem() {
        numItems++;
    }
}</pre>

As you may have guessed, the shopping cart is the object we want to save on the session.  I've implemented <code>Serializable</code> so the servlet container can save the session when you shut it down. (Tomcat does that, anyway.)

<h3>The JSP</h3>

We have just one JSP.

<pre>&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;View Shopping Cart&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;h1&gt;View Shopping Cart&lt;/h1&gt;
        &lt;p&gt;Your shopping cart currently contains ${shoppingCart.numItems} items.&lt;/p&gt;
        &lt;p&gt;&lt;a href="additem.do"&gt;Add item to cart&lt;/a&gt;&lt;/p&gt;
    &lt;/body&gt;
&lt;/html&gt;</pre>

<h3>web.xml configuration file</h3>

Nothing special here:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    version="2.5"&gt;
    
    &lt;servlet&gt;
        &lt;servlet-name&gt;front&lt;/servlet-name&gt;
        &lt;servlet-class&gt;
            org.springframework.web.servlet.DispatcherServlet
        &lt;/servlet-class&gt;
    &lt;/servlet&gt;
    &lt;servlet-mapping&gt;
        &lt;servlet-name&gt;front&lt;/servlet-name&gt;
        &lt;url-pattern&gt;*.do&lt;/url-pattern&gt;
    &lt;/servlet-mapping&gt;
&lt;/web-app&gt;</pre>

Now let's look at our Spring configuration file, which is where the aforementioned magic happens.

<h3>Spring application context</h3>

Here's our Spring config:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-2.5.xsd"
    default-autowire="byName"&gt;
    
    &lt;!-- Scan for controllers and services --&gt;
    &lt;context:component-scan base-package="ssbexample"/&gt;
    
    &lt;!-- Create a proxy to generate session-scoped shopping carts --&gt;
    &lt;bean id="shoppingCart" class="ssbexample.ShoppingCart" scope="session"&gt;
        &lt;!-- This requires CGLIB --&gt;
        &lt;aop:scoped-proxy/&gt;
    &lt;/bean&gt;
    
    &lt;!-- Maps a logical view name to a physical resource --&gt;
    &lt;bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"&gt;
        &lt;property name="prefix" value="/WEB-INF/jsp/"/&gt;
        &lt;property name="suffix" value=".jsp"/&gt;
    &lt;/bean&gt;
&lt;/beans&gt;</pre>

As I noted at the outset, we're autowiring here, so I'll just point out that the <code>shoppingCart</code> bean is automatically injected into the <code>service</code> bean (identified by the <code>@Service("service")</code> annotation inside <code>ssbexample.MyServiceImpl</code>) since I have <code>default-autowire="byName"</code> set in the Spring config.

On the <code>shoppingCart</code> bean, we have the scope set to <code>session</code>, which is probably unsurprising.  But that's not the whole story.

The interesting thing about the injection is that we're not injecting any particular shopping cart into the service bean.  Rather we're injecting a web-aware proxy into the service bean.  That's what <code>&lt;aop:scoped-proxy/&gt;</code> is for.  The proxy, being web-aware, can see individual user sessions.  So when any particular user asks for a shopping cart, the proxy grabs the shopping cart off the current session and returns it.

Note that in order to use session scope, you have to be using a web-aware Spring application context, such as <code>XmlWebApplicationContext</code>.  Otherwise there's no way for the scoped proxy to reference a current session.

<h3>Discussion</h3>

I think that this is a pretty nice tool to have in your toolbox. Using this technique, you can move the creation of session-scoped beans into the Spring application context, instead of having to create those manually in the code, and then having to place them manually on the session.  And it does make the service API cleaner, because you can avoid having to include important objects (like a shopping cart) in all the method signatures.  So I like that about session-scoped beans.

The main reservation I have is that despite appearances, there's an important sense in which using session scoping ties the code to Spring, which goes against the whole Spring philosophy of being noninvasive.  Usually service beans are designed and implemented such that a single service bean serves multiple clients, each with its own client session.  And as long as we're able to inject a web-aware shopping cart proxy into the shopping cart slot, we haven't abandoned that.  But that's the problem: if we decide to move away from Spring, we may find ourselves without easy access to a web-aware shopping cart proxy, and so we're forced to redesign the service bean in some way (such as changing the service methods to include a shopping cart parameter).

I don't think this concern invalidates the approach, but it's important to understand its ramifications.

<span class="icon stickyNote">Post migrated from my Wheeler Software site.</span>