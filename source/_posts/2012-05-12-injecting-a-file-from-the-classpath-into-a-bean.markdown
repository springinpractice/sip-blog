---
layout: post
title: "Injecting a file from the classpath into a bean"
date: 2012-05-12 23:29
comments: true
categories: [Chapter 01 – DI, Chapter 09 – Comments, Quick Tips]
---
Here's a quick tip for you.

Sometimes you need to inject a `java.io.File` from your classpath into a bean, but you don't want to have to spell out the absolute path (even in a configuration file). Never fear. It's easy:

    <bean id="tagProviderResource" class="org.springframework.core.io.ClassPathResource">
        <constructor-arg value="/htmlcleaner.xml" />
    </bean>
    
    <util:property-path id="tagProviderFile" path="tagProviderResource.file" />
    
    <bean id="tagProvider" class="org.htmlcleaner.ConfigFileTagProvider">
        <constructor-arg ref="tagProviderFile" />
    </bean>

In the configuration above, I used `ClassPathResource` to find the `htmlcleaner.xml` resource on the classpath. Then I used the handy `<util:property-path>` tag to assign the resource's `file` property its own ID. Finally, I inject the `File` using constructor injection.
