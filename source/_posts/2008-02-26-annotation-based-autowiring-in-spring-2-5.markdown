---
layout: post
title: "Annotation-based autowiring in Spring 2.5"
date: 2008-02-26 22:23:15
comments: true
categories: [Chapter 01 - DI, Tutorials]
---
This article gives a quick summary of how to do autowiring with Spring 2.5 using annotations. We cover the data and services tiers here, but not the web tier. For information on web-tier configuration, please see my article <a href="http://springinpractice.com/2008/02/26/annotation-based-mvc-in-spring-2-5/">Annotation-based MVC in Spring 2.5</a>.

To set the proper context, our goal is to get rid of manual namings and wirings in the configuration files, and the basic strategy is convention-over-configuration: name the various dependencies according to the property names so that Spring can figure out how to autowire your app.

<code>&lt;aside&gt;</code>Incidentally, it is a separate and interesting discussion as to whether autowiring is something that <em>should</em> be done in the first place.  One might argue that wiring is a configuration issue, and that adding annotations everywhere amounts to distributing the configuration across the various classes, thereby violating the principle of separation of concerns. Not to mention the fact that we don't normally want to have to rebuild the app to make a config change.  I can see that in some cases it makes sense to separate concerns (for example, I think modeling and persistence should generally be kept separate) and in other cases it doesn't (for example, code documentation should not be separated out).  Anyway I won't argue here whether autowiring is good; I'll just explain how to do it.<code>&lt;/aside&gt;</code>

Here I assume that you're using Spring MVC and Hibernate.  I'm also assuming that you already have separate beans configuration files for the web, service, and data access tiers, that they're currently not autowired (i.e. you want to migrate from a manual wiring to an autowired configuration).  Nothing essential hinges upon these assumptions&mdash;they just reflect my own configuration&mdash;and the instructions should be useful even if your configuration is somewhat different.

Here we go.

<h3>Add annotations to name your DAO, service and controller classes</h3>

Chances are good that your DAO and service classes don't have the names that name-based autowiring would require.  For example, you might define interfaces that have the "right" names, such as a <code>CategoryDao</code> interface, or a <code>ProductService</code> interface, and then your concrete implementation classes would be <code>HibernateCategoryDao</code> or <code>ProductServiceImpl</code> (or whatever), which won't autowire to the desired properties unless you have some strange and ill-advised property names.  So our first order of business is to provide the requisite names. With manual wiring, you provide this on the <code>&lt;bean&gt;</code> element using the <code>id</code> or <code>name</code> attributes.  But we're trying to eliminate said elements from our config, so that option is unavailable.  We use annotations instead.

<h4>For each DAO, add a class-level @Repository annotation.</h4>

These annotations go on the implementation class, not on the interface.

<pre>import org.springframework.stereotype.Repository;
 
@Repository("productDao")
public final class ProductDaoImpl extends AbstractHibernateDao<Product>
  implements ProductDao {

    ...
}</pre>
The <code>@Repository</code> annotation works in Spring 2.0 as well.

<h4>For each service component, add a class-level @Service annotation.</h4>

As before, these go on the implementation class, not on the interface.

<pre>import org.springframework.stereotype.Service;
 
@Service("productService")
public final class ProductServiceImpl implements ProductService {
    ...
}</pre>

Note that <code>@Service</code> is new with Spring 2.5.  So it won't work if you're using Spring 2.0 or earlier.

<h4>For each Spring MVC controller, add a class-level @Controller annotation.</h4>

Don't worry about providing a name; we won't need it.  (So this step is a little out of place in these instructions, but go ahead and do it anyway as you will need the <code>@Controller</code> annotation if you want to fully autowire the web tier.) Just do this for each controller:

<pre>import org.springframework.stereotype.Controller;
 
@Controller
public final class ProductController {
    ...
}</pre>

Like the <code>@Service</code> annotation, <code>@Controller</code> is new with Spring 2.5.

At this point we've haven't really changed anything; you still have names defined in the manual configurations in addition to the annotation-based names you've just added. Test out your app to make sure we haven't broken anything, and then move on to the next step.

<h3>Tell Spring about your annotation-based names and autowiring strategy</h3>

We have to tell Spring two things: where to find the names, and what strategy to use (viz., autowiring by name) for autowiring.

<h4>Tell Spring where to find the annotation-based names</h4>

You will need to use the <code>&lt;context:component-scan&gt;</code> element to do this. Since it's in the context namespace, you must declare that namespace and schema location in your Spring config files. Here are examples from the data access, service, and web configurations:

<pre>&lt;!-- Spring configuration for data access tier --&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd"
    default-autowire="byName"&gt;
    
    &lt;context:component-scan base-package="x.y.dao"&gt;
        &lt;context:include-filter type="annotation"
            expression="org.springframework.stereotype.Repository"/&gt;
    &lt;/context:component-scan&gt;
    
    ...
&lt;/beans&gt;
    
&lt;!-- Spring configuration for service tier --&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd"
    default-autowire="byName"&gt;
    
    &lt;context:component-scan base-package="x.y.service"&gt;
        &lt;context:include-filter type="annotation"
            expression="org.springframework.stereotype.Service"/&gt;
    &lt;/context:component-scan&gt;

    ...
&lt;/beans&gt;
    
&lt;!-- Spring configuration for web tier --&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd"
    default-autowire="byName"&gt;
    
    &lt;context:component-scan base-package="x.y.web"&gt;
        &lt;context:include-filter type="annotation"
            expression="org.springframework.stereotype.Controller"/&gt;
    &lt;/context:component-scan&gt;

    ...
&lt;/beans&gt;</pre>

(You'll of course need to provide the proper value for the base package.)

In the above, the pieces relevant to component names are the <code>xmlns:context</code> declaration, the <code>xsi:schemaLocation</code> declaration, and the <code>&lt;context:component-scan&gt;</code> configuration.  I'm telling Spring to scan the <code>x.y.dao</code> package for components that have the <code>@Repository</code> annotation, to scan the <code>x.y.service</code> package for components that have the <code>@Service</code> annotation, and to scan <code>x.y.web</code> for components that have the <code>@Controller</code> annotation.  Spring will do that, and in the case of the data access and service components, it will use the value we provided to the annotations in step 1 above as a basis for name-based autowiring.  (We didn't provide any names with the <code>@Controller</code> annotation though I would assume you can do that.  It happens that in our case we won't need it.)

You'll notice however that I've snuck in some <code>default-autowire="byName"</code> attributes in there. As you can guess, that's for autowiring, and we'll look at that in just a second.

<h4>Make sure you have public setters for each bean that you want autowired</h4>

It appears that <code>default-autowire="byName"</code> autowiring is based on public setters. (With the <code>@Autowired</code> attribute applied to fields you can avoid the setters, but if you have unit tests, you'll need setters to inject mocks, and so you may as well just keep the setters and make them public.  Weaker visibility doesn't appear to work.)

<h4>Tell Spring that you want to use name-based autowiring</h4>

The <code>default-autowire="byName"</code> attribute tells Spring that you want to use component names to automatically wire up the app.  In step 1 above we went through the process of providing correct names ("correct" here means names that match the injection property names) for DAO and service classes.  That will allow us to remove the DAO and service components from our Spring config.  In all likelihood there will be other components that you can't remove from the config, such as a Hibernate <code>SessionFactory</code> in your data access configuration or a <code>TransactionManager</code> in your services configuration. In cases like that, just make sure that the names (or IDs) that you give to the components are the ones that would be expected when doing name-based injection.  Here are some examples:

<pre>&lt;!-- Hibernate session factory --&gt;
&lt;bean id="sessionFactory"
    class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean"&gt;
    ...
&lt;/bean&gt;
 
&lt;!-- Hibernate transaction manager --&gt;
&lt;bean id="transactionManager"
    class="org.springframework.orm.hibernate3.HibernateTransactionManager"&gt;
    &lt;property name="sessionFactory" ref="sessionFactory"/&gt;
&lt;/bean&gt;</pre>

Note that using <code>default-autowire="byName"</code> allows you to avoid having to use the <code>@Autowired</code> attribute on your various dependency injection fields or setters.  You shouldn't have to use that at all if you follow these instructions.

At this point you should be able to remove the following configurations:

<ul>
<li>The DAO component definitions from your Spring data access configuration file (but don't remove the <code>SessionFactory</code>).</li>
<li>The service component definitions from your Spring service configuration file (but don't remove the <code>TransactionManager</code>).</li>
<li>The service injections from your MVC controllers in your web configuration file. Don't remove the controllers themselves for now; you still need those for defining URL mappings.  Just remove the service injections since those will now be autowired. (Note that there's a way to eliminate the MVC controller configurations as well; see <a href="http://springinpractice.com/2008/02/26/annotation-based-mvc-in-spring-2-5/">Annotation-based MVC in Spring 2.5</a> for details.)</li>
<li>The <code>sessionFactory</code> injection from your <code>TransactionManager</code> definition since that will be autowired, the <code>dataSource</code> injection from your <code>SessionFactory</code> (assuming you have an appropriate ID on the <code>DataSource</code> config), etc. In general just look through your configs for elements that will now be autowired and either comment them out or remove them.</li>
</ul>

I would recommend commenting things out first and then testing the app to make sure it still works.

<h3>Congratulations!</h3>

Your app should now be autowired.

<div class="endnote">Post migrated from my Wheeler Software site.</div>
