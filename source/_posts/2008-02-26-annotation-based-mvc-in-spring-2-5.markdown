---
link: http://springinpractice.com/2008/02/26/annotation-based-mvc-in-spring-2-5/
layout: post
title: Annotation-based MVC in Spring 2.5
date: 2008-02-26 22:36:14
comments: true
categories: [Chapter 03 - Web MVC, Tutorials]
---
Spring 2.5 introduces annotation-based configuration for MVC controllers. In this short article I'll describe at a high level what you need to do to migrate your Spring 2.0 app over to Spring 2.5, at least as far as MVC is concerned.  For information on migrating the other tiers, please see my article <a href="http://springinpractice.com/2008/02/26/annotation-based-autowiring-in-spring-2-5/">Annotation-based autowiring in Spring 2.5</a>.

First make sure you drop <code>spring-webmvc.jar</code> onto your classpath.  The <code>DispatcherServlet</code> is no longer part of <code>spring.jar</code>; it's in a separate module now.

Any given controller class can be set up in one of two ways. (And you can use both approaches in a single app.) Either the controller can handle a single action or else it can handle multiple actions. Usually a form "action", which really includes initially serving up the form and then accepting submissions of that form, would be handled with the first sort of controller, and non-form actions can be combined into a single controller, with actions being represented by methods.

Here's an extremely basic multi-action controller that handles three separate actions (mapped to methods). The actions don't do anything at all other than serve up the requested page.

<pre>package demo;
    
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
    
@Controller
public class SimpleController {
    
    @RequestMapping("/index.html")
    public void indexHandler() { }
    
    @RequestMapping("/about.html")
    public void aboutHandler() { }
    
    @RequestMapping("/admin.html")
    public void adminHandler() { }
}</pre>

Even though this is just about the simplest possible controller, there are still a few important points to highlight, especially if you're coming from earlier versions of Spring. First you will note that the controller is a POJO. It does not extend <code>AbstractController</code> or any of the other controller classes that you would have extended with Spring 2.0 or earlier. Second, notice the annotations. I've marked the controller itself with a <code>@Controller</code> annotation and the individual methods with <code>@RequestMapping</code> annotations. (I mentioned above that you can also map a controller to a single action; in that case you would attach the <code>@RequestMapping</code> annotation to the controller class itself. I'm not doing that here though.) Note also that I'm doing the URL mapping with the annotation. Pretty cool to have this option. Finally, though you can't tell from the controller, I'm using the request URL to specify the logical view name. Unless you specify otherwise, the <code>DispatcherServlet</code> will automatically assume that <code>/index.html</code> maps to the logical name "index", that <code>/admin.html</code> maps to the logical name "admin", etc. (Internally, it creates an instance of <code>DefaultRequestToViewNameTranslator</code>, though you don't see this unless you want to.) This is an example of Spring's movement toward the convention-over-configuration philosophy.

Now you need to make sure your application context config file is set up. Here's what I have:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
 
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd"&gt;
    
    &lt;context:component-scan base-package="demo"/&gt;
    
    &lt;bean id="viewResolver"
        class="org.springframework.web.servlet.view.InternalResourceViewResolver"&gt;
        &lt;property name="prefix" value="/WEB-INF/jsp/"/&gt;
        &lt;property name="suffix" value=".jsp"/&gt;
    &lt;/bean&gt;
&lt;/beans&gt;</pre>

The <code>&lt;component-scan&gt;</code> thing runs through your classes to find controllers and their annotation-based configuration. It knows which classes are controllers, as you may have guessed, because you used the <code>@Controller</code> annotation to label the controllers.  Spring also supports <code>@Repository</code> for DAO layers, <code>@Service</code> for service layers, and <code>@Component</code> as a general stereotype over <code>@Controller</code>, <code>@Repository</code> and <code>@Service</code>.) The view resolver is just like it was with Spring 2.0.

Those are the basics. Of course there's a lot more you're going to want to do; for example, you'll want to pass in request parameters, process requests, return model objects, etc.  But hopefully you'll find it useful to have a minimalistic example that gets you off the ground. Good luck!

<div class="endnote">Post migrated from my Wheeler Software site.</div>
