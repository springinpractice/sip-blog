---
link: http://springinpractice.com/2011/12/29/its-back-the-classpathscanningjaxb2marshaller/
layout: post
title: ClasspathScanningJaxb2Marshaller
date: 2011-12-29 21:03:00
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Quick Tips]
---
By default, Spring's <code>&lt;oxm:jaxb2-marshaller&gt;</code> tag uses the <a href="http://static.springsource.org/spring/docs/3.1.x/javadoc-api/org/springframework/oxm/jaxb/Jaxb2Marshaller.html">Jaxb2Marshaller</a> class. It's painful to use, though, if you have a lot of classes, because you have to enumerate them all individually. There are a couple of options here:

<ul>
<li>You can use one of the JAXB2 mechanisms (an <code>ObjectFactory</code> or an <code>jaxb.index</code> file; see <a href="http://docs.oracle.com/javase/6/docs/api/javax/xml/bind/JAXBContext.html">JAXBContext</a>).</li>
<li>You can use the marshaller's <code>classesToBeBound</code> property.</li>
</ul>

Both of these options require more work than they ought to. Here, for example, is what the <code>classesToBeBound</code> approach looks like:

<pre>&lt;oxm:jaxb2-marshaller id="marshaller"&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Database" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.DataCenter" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Environment" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Farm" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Instance" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Package" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Package$MyListWrapper" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Person" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Project" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.Region" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.model.relationship.ProjectMembership" /&gt;
    &lt;oxm:class-to-be-bound name="org.skydingo.skybase.web.jit.JitNode" /&gt;

    ... and so forth ...

&lt;/oxm:jaxb2-marshaller&gt;</pre>

Every time we add a new entity to the system, we have to go back to the list and add a corresponding entry. Blech.

About a year back I discovered a gem by <a href="https://twitter.com/#!/jwalgemoed">Jarno Walgemoed</a> called <code>ClasspathScanningJaxb2Marshaller</code>. Actually, I don't remember if that's its exact name, because his blog post no longer exists, but that's pretty close if not exactly it. At any rate, it uses Spring's classpath scanning mechanism (you know, the one that finds <code>@Component</code> and <code>@Repository</code> and so forth) to find classes annotated with <code>@XmlRootElement</code>, and then JAXB-binds them.

[EDIT: I found <a href="http://www.walgemoed.org/2010/12/jaxb2-spring-ws/">the original blog post</a> after all. -- WLW]

Here's a slightly modified version of Jarno's class, reposted with his kind permission. (Thanks Jarno!) I've used it on multiple projects and it works great.

<pre>import java.util.ArrayList;
import java.util.List;
import java.util.Set;

import javax.annotation.PostConstruct;
import javax.xml.bind.annotation.XmlRootElement;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider;
import org.springframework.core.type.filter.AnnotationTypeFilter;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;

public class ClasspathScanningJaxb2Marshaller extends Jaxb2Marshaller {
    private static final Logger log = LoggerFactory.getLogger(ClasspathScanningJaxb2Marshaller.class);
    
    private List basePackages;
    
    public List getBasePackages() { return basePackages; }
    
    public void setBasePackages(List basePackages) { this.basePackages = basePackages; }
    
    @PostConstruct
    public void init() throws Exception {
        setClassesToBeBound(getXmlRootElementClasses());
    }
    
    private Class[] getXmlRootElementClasses() throws Exception {
        ClassPathScanningCandidateComponentProvider scanner =
            new ClassPathScanningCandidateComponentProvider(false);
        scanner.addIncludeFilter(new AnnotationTypeFilter(XmlRootElement.class));
        
        List&lt;Class&gt; classes = new ArrayList&lt;Class&gt;();
        for (String basePackage : basePackages) {
            Set definitions = scanner.findCandidateComponents(basePackage);
            for (BeanDefinition definition : definitions) {
                String className = definition.getBeanClassName();
                log.info("Found class: {}", className);
                classes.add(Class.forName(className));
            }
        }
        
        return classes.toArray(new Class[0]);
    }
}</pre>
