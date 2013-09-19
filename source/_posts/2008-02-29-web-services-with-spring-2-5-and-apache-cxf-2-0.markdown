---
link: http://springinpractice.com/2008/02/29/web-services-with-spring-2-5-and-apache-cxf-2-0/
layout: post
title: Web services with Spring 2.5 and Apache CXF 2.0
date: 2008-02-29 20:19:27
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Tutorials]
---
In this tutorial I explain how to get a web service up and running using Spring 2.5 and Apache CXF 2.0, which is the <a href="http://xfire.codehaus.org/XFire+and+Celtix+Merge">combination of Celtix and XFire</a> and is considered XFire 2.0.  (I don't know what the Celtix team would say about that, but that's what the <a href="http://xfire.codehaus.org/">XFire site</a> says.)  Here I just treat the web service itself; to learn about consuming the web service using Spring and CXF, please see the article <a href="http://springinpractice.com/2008/03/10/make-web-services-transparent-with-spring-2-5-and-apache-cxf-2-0/">Make web services transparent with Spring 2.5 and Apache CXF 2.0</a>.

CXF supports both WSDL-first and Java-first web service development.  This article takes the Java-first approach.

<h3>Project setup</h3>

The first thing you'll need to do is <a href="http://incubator.apache.org/cxf/download.html">download CXF from the Apache CXF site</a>.  At the time of this writing the project is in incubator status and the latest version is 2.0.4.  That's the version I'm using.

You'll also find it useful to know about the section of the CXF user documentation that deals with <a href="http://cwiki.apache.org/CXF20DOC/writing-a-service-with-spring.html">writing a service with Spring</a>, but currently the docs describe how to integrate with Spring 2.0, and since I want to integrate with Spring 2.5, there are some differences worth highlighting along the way.

Also, the docs describe a "Hello, World" web service that just returns a string, and in this tutorial we want to go a <i>little</i> further than that and actually do a class databinding.

Here are the service-side dependencies you'll need.  Inside the distribution there is a file called <code>WHICH_JARS</code> that describes in a little more detail what the JARs are for and which ones you'll really need, if you're interested in that.  But the following is essentially the list given in the user docs.

<h4>CXF itself</h4>

<ul class="square">
    <li><code>cxf-2.0.4-incubator.jar</code></li>
</ul>

<h4>CXF dependencies</h4>

Note that for CXF 2.0.4 the documented dependencies are almost but not quite the same as the JARs that are actually included in the distribution, once again, at the time of this writing (February 2008).  Specifically the <code>stax</code>, <code>neethi</code> and <code>XMLSchema</code> JARs are not the ones listed.  Here's the corrected list for CXF 2.0.4:

<ul class="square">
<li><code>commons-logging-1.1.jar</code></li>
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

<h4>Spring dependencies</h4>

In this case ignore what's in the CXF documentation, because we're integrating with Spring 2.5 instead of Spring 2.0.  I'm going to assume that you have Spring 2.5 already set up on your web services project, including Hibernate or anything else that your web services will need on the implementation side.

Just in case you were wondering, you don't need the Spring MVC module's <code>DispatcherServlet</code> as CXF comes with its own servlet, <code>org.apache.cxf.transport.servlet.CXFServlet</code>.

OK, that should be good for basic project setup.  Let's write a simple service.  

<h3>Let's write a simple web service</h3>

Let's go basic but realistic and implement a "contact us" web service.  The idea is that lots of different applications and/or web sites need to have a web-based form to allow end users to contact an admin team (business, tech support, development, whatever) responsible for monitoring such communications.  We'll implement a web service so that multiple applications can share it easily.  There will be two operations:

<ul class="square">
    <li><b>View the list of submitted messages:</b> This allows the admin team to see what the end users are saying.</li>
    <li><b>Post a message:</b> This allows end users to ask a question, tell us how great we are, demand their money back, etc.</li>
</ul>

<h4>Create the object model</h4>

As we want to explore more than the barest "Hello, World" functionality, let's create a <code>Message</code> class that we can pass to the service and that we can get from the service.  This will allow us to play a bit with Java/XML databindings.

Here's the <code>Message</code> class:

<pre>package contactus;

import org.apache.cxf.aegis.type.java5.IgnoreProperty;

public final class Message {
    private String firstName;
    private String lastName;
    private String email;
    private String text;
    
    public Message() { }
    
    public Message(String firstName, String lastName, String email, String text) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.text = text;
    }

    public String getFirstName() { return firstName; }

    public void setFirstName(String firstName) { this.firstName = firstName; }

    public String getLastName() { return lastName; }

    public void setLastName(String lastName) { this.lastName = lastName; }
    
    @IgnoreProperty
    public String getLastNameFirstName() { return lastName + ", " + firstName; }
    
    public String getEmail() { return email; }

    public void setEmail(String email) { this.email = email; }

    public String getText() { return text; }

    public void setText(String text) { this.text = text; }
}</pre>

In real life I might have a bunch of Hibernate annotations in there (I'm a fan of annotation-based configuration), and I would create a DAO.  But as this is a tutorial, we'll keep it simple.

If you look carefully, however, you can see that I did in fact include an annotation. The <code>@IgnoreProperty</code> annotation is part of the Aegis databinding mechanism, which comes with CXF but is not the default.  (JAXB is the default.)  I ended up using Aegis instead of JAXB because JAXB was giving me trouble when I tried to return complex data types like <code>Message</code> from a web service.  Maybe it works and maybe it doesn't&mdash;I don't know&mdash;but when I plugged Aegis in it worked immediately and now I intend to use Aegis.  (More on JAXB/Aegis a little later.)  Anyway, <code>@IgnoreProperty</code> tells Aegis that I don't want <code>getLastNameFirstName()</code> to show up in the auto-generated WSDL.  It's just a read-only convenience method.

<h4>Create the service interface</h4>

Now we define a service interface.

<pre>package contactus;

import java.util.List;

import javax.jws.WebParam;
import javax.jws.WebService;

@WebService
public interface ContactUsService {
    
    List&lt;Message&gt; getMessages();
    
    void postMessage(@WebParam(name = "message") Message message);
}</pre>

The <code>@WebService</code> and <code>@WebParam</code> are <a href="http://java.sun.com/javaee/5/docs/api/javax/jws/package-summary.html">JAX-WS annotations</a>.  The first indicates that the interface defines a web service interface (we'll use it to autogenerate a WSDL) and the second allows us to use an HTTP parameter called "message" to reference the operation's argument instead of having to call it "arg0", which is the default.

<h4>Create the service implementation</h4>

Here's our simple implementation of the <code>ContactUsService</code> interface.

<pre>package contactus;

import java.util.ArrayList;
import java.util.List;

import javax.jws.WebService;

@WebService(endpointInterface = "contactus.ContactUsService")
public final class ContactUsServiceImpl implements ContactUsService {

    @Override
    public List&lt;Message&gt; getMessages() {
        List&lt;Message&gt; messages = new ArrayList&lt;Message&gt;();
        messages.add(new Message(
                "Willie", "Wheeler", "willie.wheeler@xyz.com", "Great job"));
        messages.add(new Message(
                "Dardy", "Chen", "dardy.chen@xyz.com", "I want my money back"));
        return messages;
    }

    @Override
    public void postMessage(Message message) {
        System.out.println(message);
    }
}</pre>

Here the <code>@WebService</code> annotation is marking this class as a web service implementation, and also specifying that the WSDL should be generated using the <code>ContactUsService</code> interface.

With that, we have the Java backend for the web service. Let's move on to configuration.

<h3>Configure your web service</h3>

We need to create <code>web.xml</code> and a Spring application context file.

<h4>web.xml configuration</h4>

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="services" version="2.5"&gt;
    
    &lt;context-param&gt;
        &lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;
        &lt;param-value&gt;/WEB-INF/appContext.xml&lt;/param-value&gt;
    &lt;/context-param&gt;
    &lt;listener&gt;
        &lt;listener-class&gt;
            org.springframework.web.context.ContextLoaderListener
        &lt;/listener-class&gt;
    &lt;/listener&gt;
    &lt;servlet&gt;
        &lt;servlet-name&gt;CXFServlet&lt;/servlet-name&gt;
        &lt;servlet-class&gt;
            org.apache.cxf.transport.servlet.CXFServlet
        &lt;/servlet-class&gt;
    &lt;/servlet&gt;
    &lt;servlet-mapping&gt;
        &lt;servlet-name&gt;CXFServlet&lt;/servlet-name&gt;
        &lt;url-pattern&gt;/*&lt;/url-pattern&gt;
    &lt;/servlet-mapping&gt;
&lt;/web-app&gt;</pre>

Nothing surprising here.  The main thing worth mentioning is that we're using the <code>CXFServlet</code> to process all requests, which we're assuming are all requests for web services.

As you might guess from the <code>web.xml</code> file above, we need to create a Spring application context file called <code>appContext.xml</code>.

<h4>appContext.xml configuration</h4>

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:cxf="http://cxf.apache.org/core"
    xmlns:jaxws="http://cxf.apache.org/jaxws"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
        http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd
        http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd"
    default-autowire="byName"&gt;

    &lt;!-- Load CXF modules from cxf.jar --&gt;
    &lt;import resource="classpath:META-INF/cxf/cxf.xml" /&gt;
    &lt;import resource="classpath:META-INF/cxf/cxf-extension-soap.xml" /&gt;
    &lt;import resource="classpath:META-INF/cxf/cxf-servlet.xml" /&gt;
    
    &lt;!-- Enable message logging using the CXF logging feature --&gt;
    &lt;cxf:bus&gt;
        &lt;cxf:features&gt;
            &lt;cxf:logging/&gt;
        &lt;/cxf:features&gt;
    &lt;/cxf:bus&gt;
    
    &lt;!-- The service bean --&gt;
    &lt;bean id="contactUsServiceImpl" class="contactus.ContactUsServiceImpl"/&gt;

    &lt;!-- Aegis data binding --&gt;
    &lt;bean id="aegisBean"
        class="org.apache.cxf.aegis.databinding.AegisDatabinding"
        scope="prototype"/&gt; 
    &lt;bean id="jaxws-and-aegis-service-factory"
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
 
    &lt;!-- Service endpoint --&gt;
    &lt;!-- See http://incubator.apache.org/cxf/faq.html regarding CXF + Spring AOP --&gt;
    &lt;jaxws:endpoint id="contactUsService"
            implementorClass="contactus.ContactUsServiceImpl"
            implementor="#contactUsServiceImpl"
            address="/contactus"&gt;
        &lt;jaxws:serviceFactory&gt;
            &lt;ref bean="jaxws-and-aegis-service-factory"/&gt;
        &lt;/jaxws:serviceFactory&gt;
    &lt;/jaxws:endpoint&gt;
&lt;/beans&gt;</pre>

Let's talk about <code>appContext.xml</code>.

The first three imports just import some bean definitions from the CXF JAR file.  I don't know the details and I don't think we're supposed to care about the details.

The bus config just puts logging in.  That's optional.  If you run this in a production environment you probably want to turn that off because it's verbose.

The service bean definition is just telling Spring about the service bean we wrote. We're going to put a web service endpoint in front of it.

The next piece on Aegis specifies the databinding mechanism we want to use for mapping back and forth between Java and XML.  I mentioned earlier that I tried using the default JAXB but was having problems getting it to work, so someone recommended to me to try Aegis and that worked like a charm.

Finally we define the web service endpoint.  The <code>#contactUsServiceImpl</code> piece connects the endpoint up to the service bean.  (Yes, the hash mark is required.) We use the JAX-WS/Aegis service factory that we defined earlier so we can use the Aegis databinding.

I'm not using transactions or persistence in my simple service bean.  In a realistic service you are going to have transactions and persistence.  To make CXF play nicely with AOP (and hence Spring's declarative transactions) you are going to need to include the <code>implementorClass</code> attribute as I've done.  For more details on that see the CXF FAQs at <a href="http://incubator.apache.org/cxf/faq.html">http://incubator.apache.org/cxf/faq.html</a>.

<h4>A validation annoyance in Eclipse</h4>

Though it in no way reflects a fault with Eclipse itself, if you are using Eclipse 3.3 you may be getting some annoying validation errors in the IDE, such as "<code>cvc-complex-type.2.4.c: The matching wildcard is strict, but no declaration can be found for element 'jaxws:endpoint'.</code>"  This is because the schema locations for the <code>http://cxf.apache.org/core</code> and <code>http://cxf.apache.org/jaxws</code> keys don't actually point at real XSDs, so the validator won't recognize elements from the <code>cxf</code> and <code>jaxws</code> namespaces.  (That's why I say it isn't Eclipse's fault.)  This doesn't create a problem with the actual build, but the IDE will complain as it tries to validate the Spring app context files against the missing XSDs.  I haven't yet discovered the "right" solution to this problem (I'd be delighted to have somebody tell me) but there are three workarounds I know about:

<ul class="square">
    <li>You can map the <code>http://www.springframework.org/schema/beans</code> key to the <code>http://www.springframework.org/schema/beans/spring-beans.xsd</code> location (notice that I've removed the <code>-2.5</code> part).  That gets rid of the validation errors though now you're using the Spring 2.0 beans schema.  That may be OK for you.</li>
    <li>You can turn off XML validation.  That seems pretty drastic but it's an option (and in fact it's what I've myself done).</li>
    <li>You can just ignore the errors.  Can't... bring... myself... to... do... it... .</li>
</ul>

There may be something you can do with Window &rarr; Preferences &rarr; Web and XML &rarr; XML Catalog but I didn't see it: even if you put the <code>core.xsd</code> and <code>jaxws.xsd</code> in the catalog, those XSDs in turn reference other schema locations that don't resolve to an XSD and it looks like a lot of work to me.

Anyway as I say if somebody knows the right way to handle this please let me know and I'll gladly update the article and credit you. :-)

<h3>Let 'er rip</h3>

At this point you should have a working web service. Try it out. The URLs are:

<ul class="square">
    <li><code>[path to CXFServlet]</code> - List of web services</li>
    <li><code>[path to CXFServlet]/contactus?wsdl</code> - WSDL for the Contact Us service</li>
    <li><code>[path to CXFServlet]/contactus/getMessages</code> - getMessages operation. Take a look at
    this and notice that, as promised, the individual messages don't contain the
    <code>firstNameLastName</code> property, since we told Aegis to <code>@IgnoreProperty</code>.</li>
    <li><code>[path to CXFServlet]/contactus/postMessage?message=...</code> - postMessage operation</li>
</ul>

So for example if you are running an out-of-the-box Tomcat instance then your URLs would be

<ul class="square">
    <li><code>http://localhost:8080/mywebservices</code></li>
    <li><code>http://localhost:8080/mywebservices/contactus?wsdl</code></li>
    <li><code>http://localhost:8080/mywebservices/contactus/getMessages</code></li>
    <li><code>http://localhost:8080/mywebservices/contactus/postMessage?message=...</code></li>
</ul>

replacing <code>mywebservices</code> with whatever you happened to use for your context root.

That's it!  If you want to learn how to consume web services using Spring 2.5 and CXF, please see my follow-up article <a href="http://springinpractice.com/2008/03/10/make-web-services-transparent-with-spring-2-5-and-apache-cxf-2-0/">Make web services transparent with Spring 2.5 and Apache CXF 2.0</a>.

<strong>Update (2008-09-03):</strong> Sam Brodkin was kind enough to create <a href="http://www.jroller.com/brodkin/entry/a_maven_project_for_willie">a Maven project</a> for this article. Thanks Sam!

<div class="endnote">Post migrated from my Wheeler Software site.</div>
