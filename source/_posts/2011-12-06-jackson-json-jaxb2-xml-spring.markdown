---
link: http://springinpractice.com/2011/12/06/jackson-json-jaxb2-xml-spring/
layout: post
title: Configuring Jackson to use JAXB2 annotations with Spring
date: 2011-12-06 22:17:40
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Quick Tips]
---
If you want to generate both XML and JSON views of some object, one option is to use JAXB2 annotations for the XML mapping and Jackson annotations for the JSON mapping. But that's a little bit of a nuisance, because you have to declare the what are essentially the same mappings twice. It turns out that it's possible to configure Jackson to use the JAXB2 annotations.

<h3>Maven configuration</h3>

First, make sure you have the <code>jackson-xc</code> dependency (in addition to the other Jackson dependencies) declared. This is the Maven dependency that supports using JAXB2 annotations:

<pre>&lt;dependency&gt;
    &lt;groupId&gt;org.codehaus.jackson&lt;/groupId&gt;
    &lt;artifactId&gt;jackson-xc&lt;/artifactId&gt;
    &lt;version&gt;1.9.2&lt;/version&gt;
&lt;/dependency&gt;</pre>

<h3>Spring configuration</h3>

Next, you'll need a bit of Spring configuration:
<pre>&lt;oxm:jaxb2-marshaller id="marshaller"&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Person" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Project" /&gt;
&lt;/oxm:jaxb2-marshaller&gt;

&lt;bean id="jaxbAnnIntrospector"
    class="org.codehaus.jackson.xc.JaxbAnnotationIntrospector" /&gt;
&lt;bean id="jacksonObjectMapper" class="org.codehaus.jackson.map.ObjectMapper"&gt;
    &lt;property name="serializationConfig.annotationIntrospector"
        ref="jaxbAnnIntrospector" /&gt;
    &lt;property name="deserializationConfig.annotationIntrospector"
        ref="jaxbAnnIntrospector" /&gt;
&lt;/bean&gt;

&lt;bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver"&gt;
    &lt;property name="mediaTypes"&gt;
        &lt;map&gt;
            &lt;entry key="html" value="text/html" /&gt;
            &lt;entry key="json" value="application/json" /&gt;
            &lt;entry key="xml" value="application/xml" /&gt;
        &lt;/map&gt;
    &lt;/property&gt;
    &lt;property name="viewResolvers"&gt;
        &lt;list&gt;
            &lt;bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
                p:prefix="/WEB-INF/jsp/"
                p:suffix=".jsp" /&gt;
        &lt;/list&gt;
    &lt;/property&gt;
    &lt;property name="defaultViews"&gt;
        &lt;list&gt;
            &lt;bean class="org.springframework.web.servlet.view.json.MappingJacksonJsonView"
                p:objectMapper-ref="jacksonObjectMapper"&gt;
                &lt;property name="renderedAttributes"&gt;
                    &lt;set&gt;
                        &lt;value&gt;person&lt;/value&gt;
                        &lt;value&gt;project&lt;/value&gt;
                    &lt;/set&gt;
                &lt;/property&gt;
            &lt;/bean&gt;
            &lt;bean class="org.springframework.web.servlet.view.xml.MarshallingView"
                p:marshaller-ref="marshaller" /&gt;
        &lt;/list&gt;
    &lt;/property&gt;
&lt;/bean&gt;</pre>

Note the use of the compound property names when injecting into the Jackson <code>ObjectMapper</code>. That's a little-known but useful technique for performing injections in cases where constructor and simple property injection aren't options.

See also
<ul>
	<li><a href="http://hillert.blogspot.com/2011/01/marshal-json-data-using-jackson-in.html">http://hillert.blogspot.com/2011/01/marshal-json-data-using-jackson-in.html</a></li>
	<li><a href="http://wiki.fasterxml.com/JacksonJAXBAnnotations">http://wiki.fasterxml.com/JacksonJAXBAnnotations</a></li>
</ul>