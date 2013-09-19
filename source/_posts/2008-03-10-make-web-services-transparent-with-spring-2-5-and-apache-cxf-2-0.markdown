---
layout: post
title: "Make web services transparent with Spring 2.5 and Apache CXF 2.0"
date: 2008-03-10 20:43:21
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Tutorials]
---
In our <a href="http://springinpractice.com/2008/02/29/web-services-with-spring-2-5-and-apache-cxf-2-0/">previous installment</a>, we showed how to create Java-first web services in Spring 2.5 using Apache CXF 2.0.  This time we're going to show how to consume web services, once again using Spring 2.5 and Apache CXF 2.0.

Something great about the Spring/CXF duo is that your web service clients don't know that they're web service clients.  The approach is simple: as in any Spring app, you define Java interfaces for your services and inject the service implementations into the client classes that use them (such as MVC controllers).  CXF allows you to generate dynamic proxies according to the same Java service contract&mdash;the proxies are just another service implementation&mdash;and hence your client classes have no idea that they're calling web services.

This article shows you what you need to do to get it to work.  If you haven't already done so, please see the previous article, <a href="http://springinpractice.com/2008/02/29/web-services-with-spring-2-5-and-apache-cxf-2-0/">Web services with Spring 2.5 and Apache CXF 2.0</a>, to get your web services set up.  That article describes a user comment service that I build upon in the current article.

<h3>Project setup</h3>

First we need to set up the web service client project. Here are the various dependencies involved.

<h4>Spring 2.5</h4>

These all come from the Spring 2.5 distribution.  You can use whatever Spring 2.5 libraries you need for your own app.  Here I just happen to be using Spring MVC and JSTL, though neither of those is required for Spring/CXF integration.

<ul class="square">
<li><code>jstl.jar</code> (in <code>lib/j2ee</code>)</li>
<li><code>spring.jar</code></li>
<li><code>spring-mvc.jar</code> (in <code>dist/modules</code>; the sample app uses Spring MVC)</li>
<li><code>standard.jar</code> (in <code>lib/jakarta-taglibs</code>; this is the JSTL reference implementation)</li>
</ul>

<h4>CXF and dependencies</h4>

You can get these from the CXF 2.0.4 distribution.  Whether all of these are really necessary I don't know&mdash;for example I'm not sure why we need the JavaMail library&mdash;but I'm just going by what the CXF user manual says.

<ul class="square">
<li><code>commons-logging-1.1.jar</code></li>
<li><code>cxf-2.0.4-incubator.jar</code> (this is the main CXF JAR)</li>
<li><code>geronimo-activation_1.1_spec-1.0-M1.jar</code> (or Sun's Activation jar)</li>
<li><code>geronimo-annotation_1.0_spec-1.1.jar</code> (JSR 250)</li>
<li><code>geronimo-javamail_1.4_spec-1.0-M1.jar</code> (or Sun's JavaMail jar)</li>
<li><code>geronimo-servlet_2.5_spec-1.1-M1.jar</code> (or Sun's Servlet jar)</li>
<li><code>geronimo-stax-api_1.0_spec-1.0.jar</code></li>
<li><code>geronimo-ws-metadata_2.0_spec-1.1.1.jar (JSR 181)</code></li>
<li><code>jaxb-api-2.0.jar</code></li>
<li><code>jaxb-impl-2.0.5.jar</code></li>
<li><code>jaxws-api-2.0.jar</code></li>
<li><code>neethi-2.0.2.jar</code></li>
<li><code>saaj-api-1.3.jar</code></li>
<li><code>saaj-impl-1.3.jar</code></li>
<li><code>wsdl4j-1.6.1.jar</code></li>
<li><code>wstx-asl-3.2.1.jar</code></li>
<li><code>XmlSchema-1.3.2.jar</code></li>
<li><code>xml-resolver-1.2.jar</code></li>
</ul>

<h4>Aegis dependencies</h4>

In addition to the above, you will need to add <code>jdom-1.0.jar</code> since Aegis databinding uses it. 

<h4>Client library dependencies</h4>

In the previous article we created three classes in the <code>contactus</code> package: <code>ContactUsService</code>, <code>ContactUsServiceImpl</code>, and <code>Message</code>.  You will need to JAR <code>ContactUsService</code> (which is a service interface) and <code>Message</code> (which is a domain model class) and add the JAR as a dependency for the client application.  You do not need to include <code>ContactUsServiceImpl</code> since that's the backend for the web service; it's not part of the interface.  On the client side we will see that CXF will create a web service proxy according to the <code>ContactUsService</code> interface, and it will use <code>Message</code> for marshalling and unmarshalling.

With the dependencies in place, we'll now create a simple client that can view user messages using the web service we created the last time.  We're not going to treat the "post user message" operation in this article though it works in exactly the same way (which is why we're not going to treat it).

<h3>Creating the client application</h3>

This sample app is based on Spring MVC.  For that we'll need to create a couple of MVC controllers, a couple of JSPs, a <code>web.xml</code> file, and a Spring application context file.

<h4>Spring MVC controller for viewing messages</h4>

First we'll create a Spring MVC controller called <code>myapp.ViewMessagesController</code>.

<pre>package myapp;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;

import contactus.ContactUsService;

@Controller
public final class ViewMessagesController {
    private ContactUsService contactUsService;
    
    public void setContactUsService(ContactUsService contactUsService) {
        this.contactUsService = contactUsService;
    }
    
    @RequestMapping("/viewmessages.do")
    public ModelMap viewMessages() {
        return new ModelMap("messages", contactUsService.getMessages());
    }
}</pre>

Since this is an article about Spring/CXF integration and not about Spring MVC, I won't go into the details of Spring MVC or annotation-based configuration.  If however you are interested in learning more, please see my article <a href="spring-mvc-annotations.html">Annotation-Based MVC in Spring 2.5</a>.

At any rate, the code is easy enough to understand.  We have a dependency injection method <code>setContactUsService</code>.  Any implementation of <code>ContactUsService</code> could go in there, including the one we wrote in the previous article, <code>contactus.ContactUsServiceImpl</code>.  In this case, though, we're going to use a CXF-generated dynamic proxy as shown below.

The <code>viewMessages</code> method simply grabs a list of messages from the <code>ContactUsService</code> and puts it in a model that the JSP will be able to see.

<h4>JSP for viewing messages</h4>

You will need to put this file at <code>/WEB-INF/jsp/viewmessages.jsp</code> in order for the request mapping to work as specified in <code>ViewMessagesController</code> above and <code>myapp-servlet.xml</code> below.  For those who are unfamiliar with Spring's annotation-based MVC configuration, the <code>@RequestMapping("/viewmessages.do")</code> annotation maps the given path to the <code>viewMessages</code> method, and the view resolver in <code>myapp-servlet.xml</code> tells Spring to look inside <code>/WEB-INF/jsp/</code> for the corresponding JSP.

<pre>&lt;%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %&gt;
&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;View Messages&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;h1&gt;View Messages&lt;/h1&gt;
        &lt;ul&gt;
            &lt;c:forEach var="message" items="${messages}"&gt;
                &lt;li&gt;${message.lastNameFirstName} (${message.email}): ${message.text}&lt;/li&gt;
            &lt;/c:forEach&gt;
        &lt;/ul&gt;
    &lt;/body&gt;
&lt;/html&gt;</pre>

This just displays a list of user messages.  I'm using JSTL and JSTL EL.  The <code>${messages}</code> reference in the <code>forEach</code> tag refers to the list of messages that I placed in the <code>ModelMap</code> in <code>ViewMessagesController.viewMessages</code> above.  Spring takes care of making the contents of the <code>ModelMap</code> available to the JSP.

<h4>web.xml</h4>

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="ws" version="2.5"&gt;
    
    &lt;servlet&gt;
        &lt;servlet-name&gt;myapp&lt;/servlet-name&gt;
        &lt;servlet-class&gt;org.springframework.web.servlet.DispatcherServlet&lt;/servlet-class&gt;
        &lt;load-on-startup&gt;1&lt;/load-on-startup&gt;
    &lt;/servlet&gt;
    &lt;servlet-mapping&gt;
        &lt;servlet-name&gt;myapp&lt;/servlet-name&gt;
        &lt;url-pattern&gt;*.do&lt;/url-pattern&gt;
    &lt;/servlet-mapping&gt;
&lt;/web-app&gt;</pre>

This just sets up the Spring MVC <code>DispatcherServlet</code> and tells the servlet container that the <code>*.do</code> extension mapping goes to the <code>DispatcherServlet</code>.

Now let's dig into the guts of it&mdash;the Spring application context.

<h3>Creating the Spring application context</h3>

This is where all the good stuff is: in the Spring application context.

<h4>Spring application context: myapp-servlet.xml</h4>

Put this file at <code>/WEB-INF/myapp-servlet.xml</code> so that it's where <code>DispatcherServlet</code> expects to find it. The <code>myapp-</code> part needs to match the value specified for <code>&lt;servlet-name&gt;</code> above.  (There's a way to change this but I'm not worried about that for this sample app.)

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd"&gt;

    &lt;!-- Configure CXF to use Aegis data binding instead of JAXB --&gt;
    &lt;bean id="aegisBean"
          class="org.apache.cxf.aegis.databinding.AegisDatabinding"
          scope="prototype"/&gt;
    &lt;bean id="jaxwsAndAegisServiceFactory"
          class="org.apache.cxf.jaxws.support.JaxWsServiceFactoryBean"
          scope="prototype"&gt; 
        &lt;property name="dataBinding" ref="aegisBean"/&gt;
        &lt;property name="serviceConfigurations"&gt;
          &lt;list&gt;
            &lt;bean class="org.apache.cxf.jaxws.support.JaxWsServiceConfiguration"/&gt;
            &lt;bean class="org.apache.cxf.aegis.databinding.AegisServiceConfiguration"/&gt;
            &lt;bean class="org.apache.cxf.service.factory.DefaultServiceConfiguration"/&gt; 
          &lt;/list&gt;
        &lt;/property&gt;
    &lt;/bean&gt;

    &lt;!-- Factory to create the dynamic proxy --&gt;
    &lt;bean id="contactUsFactory" class="org.apache.cxf.jaxws.JaxWsProxyFactoryBean"&gt;
        &lt;property name="serviceClass" value="contactus.ContactUsService"/&gt;
        &lt;property name="address"
                  value="http://localhost:8080/cxf-service-example/contactus"/&gt;
        &lt;property name="serviceFactory" ref="jaxwsAndAegisServiceFactory"/&gt;
    &lt;/bean&gt;
    
    &lt;!-- Web service dynamic proxy --&gt;
    &lt;bean id="contactUsService"
          class="contactus.ContactUsService"
          factory-bean="contactUsFactory"
          factory-method="create"/&gt;
    
    &lt;!-- Controllers --&gt;
    &lt;bean class="myapp.ViewMessagesController"&gt;
        &lt;property name="contactUsService" ref="contactUsService"/&gt;
    &lt;/bean&gt;
    
    &lt;!-- View resolvers --&gt;
    &lt;bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver"&gt;
        &lt;property name="prefix" value="/WEB-INF/jsp/"/&gt;
        &lt;property name="suffix" value=".jsp"/&gt;
    &lt;/bean&gt;
&lt;/beans&gt;</pre>

There's lots to talk about here.  Let's do it bean-by-bean:

<b>aegisBean:</b> CXF supports different databinding mechanisms (i.e., mechanisms for mapping back and forth between Java and XML). The default is JAXB but I was never able to get that to work for complex data types like <code>contactus.Message</code>.  So instead I used Aegis (which is bundled with CXF) and it works great.

<b>jaxwsAndAegisServiceFactory:</b> This factory creates service models either based on a service's WSDL or else based on the structure of the service class.  These service models are in turn used by the service proxy factory, which creates your client-side web-service-aware dynamic proxies.  Anyway, you don't have to worry too much about <code>jaxwsAndAegisServiceFactory</code>; the only reason we're including it is that we need to be able to tell it to use Aegis databinding.

<b>contactUsFactory:</b> This is a factory that generates the dynamic proxies we've been talking about.  These proxies implement the <code>contactus.ContactUsService</code> interface, and this is what gives us the transparency referenced in the title of this article: the client application has no idea that it's working with web services at all (other than in the configuration).  If you wanted to, you could collapse the client app and the <code>contactus.ContactUsServiceImpl</code> backend and remove the web service altogether.  That's part of the beauty of Spring (and of Java interfaces and dynamic proxies).

We specify not only the service class but also the web service endpoint address and the service factory.  Again the only reason we have to deal with the service factory at all is that we're using the non-default Aegis databinding.  If you're able to get it working with JAXB then more power to you.  (And I'd be interested in hearing about it!)

<strong>IMPORTANT:</strong> You will need to set the host, port and context path in the <code>address</code> property to match your web service deployment address.  The one in the config file is the one I'm using but you're probably using at least a different context path.

The other beans are just Spring MVC beans.  Nothing we need to worry about here.

We're now ready to try it out!

<h3>Try it out</h3>

Your application's view messages functionality should now work. Start up both your web service and your application.  Then point your browser to

<code>http://localhost:8080/cxf-client-example/viewMessages.do</code>

<strong>(well, substitute in your host, port and context path!)</strong> and you should see the pair of hardcoded messages that we created in the last article:

<div style="margin:20px 0"><img src="http://wheelersoftware.s3.amazonaws.com/articles/spring-cxf-consuming-web-services/browser.png" alt="Screenshot" /></div>

If you got the screen above, then it's working!  Your application is pulling the messages from a web service, and transparently so.  If you decide you don't want to use web services, no problem: just pull the web service out and wire the service bean right into your MVC controller or other client class.

I hope you found this useful.  As always let me know if you have comments, questions or corrections.

<h3>Resources</h3>

<ul class="square">
<li><a href="http://wheelersoftware.s3.amazonaws.com/articles/spring-cxf-consuming-web-services/cxf-service-example.zip">Service source code (JARs not included)</a></li>
<li><a href="http://wheelersoftware.s3.amazonaws.com/articles/spring-cxf-consuming-web-services/cxf-client-example.zip">Client source code (JARs not included)</a></li>
</ul>

<strong>Update (2008-09-03):</strong> Sam Brodkin was kind enough to create <a href="http://www.jroller.com/brodkin/entry/a_maven_project_for_willie">a Maven project</a> for this article. Thanks Sam!

<div class="endnote">Post migrated from my Wheeler Software site.</div>
