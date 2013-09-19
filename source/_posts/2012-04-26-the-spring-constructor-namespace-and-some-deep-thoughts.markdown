---
link: http://springinpractice.com/2012/04/26/the-spring-constructor-namespace-and-some-deep-thoughts/
layout: post
title: The Spring constructor namespace
date: 2012-04-26 02:14:51
comments: true
categories: [Chapter 01 - DI, Quick Tips]
---
You know the <code>p</code> namespace? It's the one that cleans up your Spring configuration files by providing a shorthand for injecting beans and values:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation=
       "http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"&gt;
    
    &lt;bean id="sessionFactory"
        class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean"
        p:dataSource-ref="dataSource"
        p:packagesToScan="com.springinpractice.ch10.model" /&gt;

    ... other beans ...

&lt;/beans&gt;</pre>

Did you know that Spring 3.1 introduces an analogous namespace for constructor injection? Yup&mdash;it's the <code>c</code> namespace:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation=
       "http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd"&gt;
        
    &lt;bean id="connectionFactoryLocator"
        class="org.springframework.social.connect.support.ConnectionFactoryRegistry"&gt;
        &lt;property name="connectionFactories"&gt;
            &lt;list&gt;
                &lt;bean class="org.springframework.social.github.connect.GitHubConnectionFactory"
                    c:clientId="${gitHub.clientId}"
                    c:clientSecret="${gitHub.clientSecret}" /&gt;
            &lt;/list&gt;
        &lt;/property&gt;
    &lt;/bean&gt;
    
    &lt;bean id="usersConnectionRepository"
        class="org.springframework.social.connect.jdbc.JdbcUsersConnectionRepository"
        c:dataSource-ref="dataSource"
        c:connectionFactoryLocator-ref="connectionFactoryLocator"
        c:textEncryptor-ref="textEncryptor" /&gt;
    
    ... other beans ...

&lt;/beans&gt;</pre>

It works pretty much the same way, but for constructors. You can inject values, or else you can inject references using the <code>-ref</code> suffix.

With <code>p</code>, the XML element name comes from the bean property name, but with constructors that's not available. Instead, for <code>c</code> we use the name of the constructor parameter itself. So for <code>JdbcUsersConnectionRepository</code> we're calling a constructor that looks like this:

<code>public JdbcUsersConnectionRepository(DataSource dataSource, ConnectionFactoryLocator connectionFactoryLocator, TextEncryptor textEncryptor)</code>

where it's the parameter names themselves (rather than the parameter types) driving the XML element names.

<h3>But keep this in mind...</h3>

The <code>c</code> namespace is great, but there's something you may not have considered. Because the XML element names come from the constructor parameter names, we have to compile the app in debug mode if we want to use this feature (otherwise the local variable names aren't in the class files), and we want to pay attention to the parameter names we choose. So, for instance, <code>ds</code>, <code>cfl</code> and <code>te</code> wouldn't be very good constructor parameter names.

Note that Spring (especially Spring Web MVC) has been using parameter names in this fashion for a long time, so that's nothing new. The <code>@PathVariable</code> and <code>@RequestParam</code> annotations, for example, default to using parameter names when explicit names aren't specified. But in the case of the <code>c</code> namespace, a bean that supports <code>c</code>-based injection ought to publish the constructor parameter names, and they become in effect part of the API.

<h3>Update</h3>

I've had a bit of a change of heart regarding the constructor namespace. Here's why I think <a href="http://springinpractice.com/2012/05/07/springs-constructor-namespace-is-a-bad-idea/">the Spring constructor namespace is a bad idea</a>.
