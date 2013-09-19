---
layout: post
title: "Supporting XML and JSON web service endpoints in Spring 3.1 using @ResponseBody"
date: 2012-02-22 02:21:32
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Quick Tips]
---
In <a href="http://springinpractice.com/2011/12/06/jackson-json-jaxb2-xml-spring/" title="Configuring Jackson to use JAXB2 annotations with Spring">an earlier post</a> I explained how to avoid parallel JAXB2/Jackson annotations when supporting both XML and JSON web service endpoints. Basically the idea was to configure the Jackson <code>ObjectMapper</code> to understand JAXB2 annotations.

<h3>The problem</h3>

One of the things that was a little unsatisfactory about that post was that I was using views to generate the XML and JSON representations. We can do that, of course, but a more direct approach to generating the desired data payloads is to use <code>@ResponseBody</code> in conjunction with Spring's <code>HttpMessageConverter</code>s. The idea here is that handler methods simply return the object to be mapped (whether to XML or to JSON), and then HTTP message converters sitting on the app context map the object to XML or JSON. Correspondingly, there are HTTP message converters for both JAXB2 and Jackson (among several others). I say this is more direct because there's no need to involve a model or a view at all: the controller simply returns the object and the converters do the rest.

In Spring 3.0 this was somewhat challenging to pull off, because both the JAXB2 message converter and the Jackson message converter are able to map the object, and so whichever message converter appears first in the list (the JAXB2 converter, it seems) would always map the object. So we ended up either having to treat JSON as a special case (invoke the <code>ObjectMapper</code> directly), or else just use views after all.

In Spring 3.1 things are a lot better: we can use the <code>@ResponseBody</code> and HTTP message converter approach quite cleanly, owing to an enhancement to <code>@RequestMapping</code> and also to an enhancement to the <code>&lt;mvc:annotation-driven&gt;</code> configuration.

Let's see how it works.

<h3>Implementing the controller</h3>

We'll look at the controller code first:

<pre>@RequestMapping(
    value = "/{id}.json",
    method = RequestMethod.GET,
    produces = "application/json")
@ResponseBody
public Person getDetailsAsJson(@PathVariable Long id) {
    return personRepo.findOne(id);
}

@RequestMapping(
    value = "/{id}.xml",
    method = RequestMethod.GET,
    produces = "application/xml")
@ResponseBody
public Person getDetailsAsXml(@PathVariable Long id) {
    return personRepo.findOne(id);
}</pre>

To use the <code>@ResponseBody</code> approach, we obviously need to annotate the handler methods with that annotation, so that's what we've done here. The other thing we do here is return the actual entity (in this case, a <code>Person</code>) from the method instead of returning a logical view name.

We've also specified the extension (either <code>.json</code> or <code>.xml</code>) in the value to route requests correctly.

So far, everything we&#039;ve described was available in Spring 3.0. But notice the new <code>produces</code> element, introduced with Spring 3.1, in the <code>@RequestMapping</code> annotation. This element has two effects. First, it excludes requests with <code>Accepts</code> headers incompatible with the specified media type. Second, it ensures that the generated response is consistent with the specified media type. It's this second effect that we care about, because this is what allows Spring to figure out which HTTP message converter to use, even when we're using JAXB2 annotations for both XML and JSON mapping.

<h3>Configuration</h3>

We need some configuration too. Here are the relevant bits. Be sure you're using Spring 3.1, that you've declared the <code>oxm</code> and <code>mvc</code> namespaces, etc.

<pre>&lt;oxm:jaxb2-marshaller id="marshaller"&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Person" /&gt;
&lt;/oxm:jaxb2-marshaller&gt;
    
&lt;bean id="jaxbAnnIntrospector" class="org.codehaus.jackson.xc.JaxbAnnotationIntrospector" /&gt;
&lt;bean id="jacksonObjectMapper" class="org.codehaus.jackson.map.ObjectMapper"&gt;
    &lt;property name="serializationConfig.annotationIntrospector" ref="jaxbAnnIntrospector" /&gt;
    &lt;property name="deserializationConfig.annotationIntrospector" ref="jaxbAnnIntrospector" /&gt;
&lt;/bean&gt;

&lt;mvc:annotation-driven&gt;
    &lt;mvc:message-converters&gt;
        &lt;bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"
            p:objectMapper-ref="jacksonObjectMapper" /&gt;
    &lt;/mvc:message-converters&gt;
&lt;/mvc:annotation-driven&gt;</pre>

The JAXB2 marshaller and Jackson <code>ObjectMapper</code> are the same as they were in my earlier post. The thing that's different is the <code>&lt;mvc:annotation-driven&gt;</code> definition, and specifically, my inclusion of the new Spring 3.1 <code>&lt;mvc:message-converters&gt;</code> configuration. This allows us to define converters that override the defaults. Here I want to override the default Jackson converter with one that knows about my JAXB2-enabled <code>ObjectMapper</code>.

You don't need to include XML or JSON views anymore. If you don't have anything other than the normal JSP view, you can probably get rid of the <code>ContentNegotiatingViewResolver</code> entirely.

<h3>The end result</h3>

The result is that we can now generate XML and JSON payloads in a simple and consistent fashion. There's no more need for models and views here (at least with respect to generating XML and JSON), and there's no need to invoke <code>ObjectMapper</code>s directly or anything like that.
