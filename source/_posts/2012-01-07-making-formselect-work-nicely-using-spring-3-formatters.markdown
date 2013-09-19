---
link: http://springinpractice.com/2012/01/07/making-formselect-work-nicely-using-spring-3-formatters/
layout: post
title: Making form:select work nicely using Spring 3 Formatters
date: 2012-01-07 02:51:10
comments: true
categories: [Chapter 04 - Web forms, Quick Tips]
---
When you're creating a web form, it's often the case that you want to give the user a way to connect one entity up to another entity. This post explains how to do this in a nice, clean way using Spring Formatters.

<h3>A real-life example</h3>

To make things concrete, let's use a real-life example. I'm working on an open source CMDB called <a title="Zkybase GitHub site" href="https://github.com/williewheeler/zkybase">Zkybase</a> that has entities such as farms, environments and data centers. When the user creates or edits a farm, he needs to specify the environment in which the farm lives (e.g., Development, Test, Production, etc.) and also the data center (e.g., US West DC 1, US West DC 2, etc.). We want the user to select an environment and a data center using dropdowns, like so:

<a href="http://springinpractice.com/wp-content/uploads/2012/01/new_farm2.png"><img class="alignnone size-full wp-image-515" title="New farm form" src="http://springinpractice.com/wp-content/uploads/2012/01/new_farm2.png" alt="" width="526" height="312" /></a>

For edits, we obviously want the dropdowns to be preset to their current values. We want validation: in our example, all three fields are required. For both edits and creates, when the user submits an invalid farm (e.g., missing name), we want the environment and data center to be preset to whatever value the user just submitted. Standard stuff.

Let's look at three different approaches to solving this in Spring. The first two are ugly and the last one is clean. (If you're impatient, feel free to skip right to the third approach.)
<h3>Approach #1 (ugly): ad hoc ID fields</h3>
One approach is to create special fields on the <code>Farm</code> entity to hold the environment and data center IDs, and then write custom code in the controller to map these to <code>Environment</code> and <code>DataCenter</code> instances. Here's how such a <code>Farm</code> might look:
<pre>public class Farm {
    private Long id;
    private String name;
    private Environment environment;
    private DataCenter dataCenter;
    private Long environmentId;
    private Long dataCenterId;

    ... getters and setters for id, name, environment and dataCenter ...

    @NotNull
    @Transient
    @XmlTransient
    public Long getEnvironmentId() { return environmentId; }

    public void setEnvironmentId(Long id) { this.environmentId = id; }

    @NotNull
    @Transient
    @XmlTransient
    public Long getDataCenterId() { return dataCenterId; }

    public void setDataCenterId(Long id) { this.dataCenterId = id; }
}</pre>
And the form code looks something like this (I'm suppressing some details just for clarity's sake):
<pre>&lt;form:select path="environmentId"&gt;
    &lt;form:option value="" label="-- Choose one--" /&gt;
    &lt;form:options items="${environmentList}" itemValue="id" itemLabel="name" /&gt;
&lt;/form:select&gt;
&lt;form:errors path="environmentId"&gt;
    &lt;span class="help-inline"&gt;&lt;form:errors path="environmentId" /&gt;&lt;/span&gt;
&lt;/form:errors&gt;</pre>
The approach works, but it's stinky for at least a couple of reasons:
<ul>
	<li>It dirties up the <code>Farm</code> entity. We already have <code>Environment</code> and <code>DataCenter</code> properties, and each of those has an ID, so it's redundant and annoying to have to include extra getters and setters to handle the IDs. If we're doing ORM or OXM, it becomes very clear from the <code>@Transient</code> and <code>@XmlTransient</code> annotations that these extra ID properties aren't really part of the entity at all; they're purely supporting data transfer.</li>
	<li>It forces us to write custom code in the controller to map the IDs to an <code>Environment</code> and a <code>DataCenter</code> so we can, e.g., save it in the database using Hibernate or whatever.</li>
</ul>

Now let's see a closely related approach that's a step in the right direction, but still stinky.

<h3>Approach #2 (still ugly): use the referenced entities' ID properties</h3>

Instead of using redundant ID properties, we can just leave the <code>Farm</code> alone (meaning it has an ID, a name, an environment and a data center, and no extra ID properties) and just point the form at the environment's and data center's IDs:

<pre>&lt;form:select path="environment.id"&gt;
    &lt;form:option value="" label="-- Choose one--" /&gt;
    &lt;form:options items="${environmentList}" itemValue="id" itemLabel="name" /&gt;
&lt;/form:select&gt;
&lt;form:errors path="environment.id"&gt;
    &lt;span class="help-inline"&gt;&lt;form:errors path="environment.id" /&gt;&lt;/span&gt;
&lt;/form:errors&gt;</pre>

Spring is fine with complex properties like this. And so this almost works. But there are once again a couple of issues:

<ul>
	<li>Stylistically, it's a bit inelegant to treat reference-backed properties in a special way. It would be cleaner to deal with <code>environment</code> and <code>dataCenter</code> instead of <code>environment.id</code> and <code>dataCenter.id</code>. Minor issue, but still worth considering.</li>
	<li>More seriously, validation (at least JSR-303 validation) doesn't work properly anymore. Referencing <code>environment.id</code> in the form causes Spring to create an <code>Environment</code> instance automatically, even if the user picks "-- Choose one --" with ID = "". So the <code>environment</code> and <code>dataCenter</code> properties won't ever be null. And we can't put <code>@NotNull</code> on the environment and data center IDs, because they're allowed (indeed, expected) to be null when creating new ones. So you end up having to write custom code to make sure that the IDs are whatever you want to see. Blech.</li>
</ul>

OK, those are some approaches that aren't very clean. Fortunately Spring 3 has some features that clean things up.

<h3>Approach #3 (good approach): use Formatters</h3>

The best practice approach in Spring 3 is to use so-called <code>Formatters</code>. As this blog post is already pretty long, let's just jump right into the code. If you want to see more code details, check out the actual code at the <a title="Zkybase GitHub site" href="https://github.com/williewheeler/zkybase">Zkybase Github site</a>.

First, our entities don't have any ad hoc ID properties like we saw in approach #1 above. <strong>But you do need to implement <code>equals()</code> for your entities, or else form prepopulation won't happen.</strong>

Next, here's what the form looks like:

<pre>&lt;form:select path="environment"&gt;
    &lt;form:option value="" label="-- Choose one--" /&gt;
    &lt;form:options items="${environmentList}" itemValue="id" itemLabel="name" /&gt;
&lt;/form:select&gt;
&lt;form:errors path="environment"&gt;
    &lt;span class="help-inline"&gt;&lt;form:errors path="environment" /&gt;&lt;/span&gt;
&lt;/form:errors&gt;</pre>

(Same thing for the <code>dataCenter</code> property.) Notice that we're not messing around with IDs at all, other than where we're telling the <code>&lt;form:options&gt;</code> tag which property to use for the option value, which is perfectly fine and legitimate.

Next we need to look at the controller. In earlier versions of Spring we would have had to register some JavaBeans <code>PropertyEditor</code>s in an <code>@InitBinder</code> method, but we don't have to do that anymore. All we have to do is make sure we're prepared to accept a <code>Farm</code> in our handler method, that we annotate it with <code>@Valid</code>, etc.:

<pre>@RequestMapping(value = "/{id}", method = RequestMethod.PUT)
public String putEditForm(
        @PathVariable Long id,
        @ModelAttribute("formData") @Valid Farm formData,
        BindingResult result,
        Model model) {

    ...
}</pre>

The method body doesn't matter so much for our current purpose--we can check for validity using <code>result.hasErrors()</code>, save the data if it's valid, return the invalid form if it's not, or whatever. The main thing is that the <code>Farm</code> will come populated with an <code>Environment</code> and a <code>DataCenter</code> (both having the right IDs set) if the user chose them, and they'll be null if the user didn't:

<a href="http://springinpractice.com/wp-content/uploads/2012/01/farm_errors2.png"><img class="alignnone size-full wp-image-539" title="Farm with validation errors" src="http://springinpractice.com/wp-content/uploads/2012/01/farm_errors2.png" alt="" width="520" height="304" /></a>

To make the magic work, we need to implement <code>Formatter</code>s to convert our environments and data centers back and forth to IDs. Regarding <code>Formatter</code>s, I'll let you read about them in the <a href="http://static.springsource.org/spring/docs/current/spring-framework-reference/html/validation.html#format">Spring Reference Documentation</a>; here I'll give some code examples. Here's the <code>EnvironmentFormatter</code>:

<pre>package org.zkybase.formatter;

import java.text.ParseException;
import java.util.Locale;
import org.zkybase.model.Environment;
import org.springframework.format.Formatter;
import org.springframework.stereotype.Component;

@Component
public class EnvironmentFormatter implements Formatter&lt;Environment&gt; {

    @Override
    public String print(Environment environment, Locale locale) {
        return environment.getId().toString();
    }

    @Override
    public Environment parse(String id, Locale locale) throws ParseException {

        // IMPORTANT: This approach works only if your equals() method doesn't compare fields
        // beyond the ID. If it does, then you'll need those fields set too. Consider simply
        // loading the entity from the database.
        Environment environment = new Environment();
        environment.setId(Long.parseLong(id));
        return environment;
    }
}</pre>

One more thing we have to do is configure the formatters in our Spring configuration. Here's what that looks like (Spring 3.0.6+; I'm suppressing the namespace declarations):

<pre>&lt;context:component-scan base-package="org.zkybase.formatter" /&gt;

&lt;bean id="conversionService"
    class="org.springframework.format.support.FormattingConversionServiceFactoryBean"&gt;
    &lt;property name="formatters"&gt;
        &lt;set&gt;
            &lt;ref bean="dataCenterFormatter" /&gt;
            &lt;ref bean="environmentFormatter" /&gt;
        &lt;/set&gt;
    &lt;/property&gt;
&lt;/bean&gt;

&lt;mvc:annotation-driven conversion-service="conversionService" /&gt;</pre>

This tells Spring Web MVC to use the conversion service both when rendering the form and when processing submitted form data. We won't go into all the gory details here since they're not necessarily important to making the whole thing work, but the formatter's <code>print()</code> method is the one that handles form rendering, and the formatter's <code>parse()</code> method handles turning the IDs in the HTML into entities in the Java controller.

<strong>Quick tip regarding Firefox.</strong> Before I close, I wanted to offer a quick tip. When you're implementing and troubleshooting this stuff, be aware that <a href="http://stackoverflow.com/questions/4862606/when-using-html-select-tag-changed-selected-value-not-displayed-in-firefox">Firefox has a feature where it remembers your most recently submitted form values when you hit the refresh button</a> instead of resetting the form to its default settings. The feature helps users avoid data loss, but for developers it's a pain because you have to reload the page from the actual address bar (click in there and hit Enter) instead of refreshing the page. I didn't know this and the form was not responding to my code changes in the way I expected.
