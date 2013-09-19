---
link: http://springinpractice.com/2012/04/24/autogenerate-daos-and-queries-using-spring-data-jpa/
layout: post
title: Dynamic DAOs and queries using Spring Data JPA
date: 2012-04-24 23:18:27
comments: true
categories: [Chapter 02 - Data, Tutorials]
---
For a long time, creating a DAO layer in Spring has been a largely manual process:

<ol>
	<li>Create a base generic DAO interface.</li>
	<li>Create a generic abstract DAO implementation with general-purpose CRUD methods and common queries (e.g., <code>findAll</code>()).</li>
	<li>For each DAO we want, extend the base DAO interface with an entity-specific interface (e.g., <code>CustomerDao</code>).</li>
	<li>For each DAO we want, extend the abstract DAO class with an entity-specific concrete class (e.g., <code>HibernateCustomerDao</code>).</li>
</ol>

Items 1 and 2 amount to creating a homegrown DAO framework, and items 3 and 4 amount to using it to implement DAOs.

Now there's a better way to do things. The <a href="http://www.springsource.org/spring-data">Spring Data</a> family of projects provides a ready-made DAO framework. There are different projects, such as <a href="http://www.springsource.org/spring-data/jpa">Spring Data JPA</a>, <a href="http://www.springsource.org/spring-data/neo4j">Spring Data Neo4j</a> and <a href="http://www.springsource.org/spring-data/mongodb">Spring Data MongoDB</a>. Something they all have in common is that they provide framework code so we don't have to implement it ourselves.

Moreover, Spring Data is able to generate concrete DAO implementations and custom queries automatically. So even step 4 above goes away in many cases. With Spring Data JPA you can create DAO tiers by defining interfaces.

In this post we'll learn how to use Spring Data JPA to clean up our DAO tier. Let's get the POM and configuration out of the way first. Then we'll get into the actual repository code. We won't get into the details of mapping actual entities (via JPA annotations or otherwise) as that's outside the scope of what we want to cover here.

<h3>POM</h3>

First we need to choose which JPA provider and which package versions we want to work with. For the JPA provider, we'll use Hibernate. For the package versions, we'll use Spring 3.1.1, Spring Data JPA 1.0.3 and Hibernate 4.1.1 since those are current at the time I'm writing this.

Here are the relevant Maven dependencies for <code>pom.xml</code>:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;project ...&gt;

    ...

    &lt;!-- Use whatever versions make sense for your project. --&gt;
    &lt;properties&gt;
        &lt;hibernate.version&gt;4.1.1.Final&lt;/hibernate.version&gt;
        &lt;spring.version&gt;3.1.1.RELEASE&lt;/spring.version&gt;
        &lt;spring.data.version&gt;1.0.3.RELEASE&lt;/spring.data.version&gt;
    &lt;/properties&gt;

    &lt;dependencies&gt;

        ... standard Spring dependencies (beans, context, core, etc.) ...

        &lt;dependency&gt;
            &lt;groupId&gt;org.springframework.data&lt;/groupId&gt;
            &lt;artifactId&gt;spring-data-jpa&lt;/artifactId&gt;
            &lt;version&gt;${spring.data.version}&lt;/version&gt;
        &lt;/dependency&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;org.hibernate&lt;/groupId&gt;
            &lt;artifactId&gt;hibernate-core&lt;/artifactId&gt;
            &lt;version&gt;${hibernate.version}&lt;/version&gt;
        &lt;/dependency&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;org.hibernate&lt;/groupId&gt;
            &lt;artifactId&gt;hibernate-entitymanager&lt;/artifactId&gt;
            &lt;version&gt;${hibernate.version}&lt;/version&gt;
        &lt;/dependency&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;org.hibernate&lt;/groupId&gt;
            &lt;artifactId&gt;hibernate-ehcache&lt;/artifactId&gt;
            &lt;version&gt;${hibernate.version}&lt;/version&gt;
        &lt;/dependency&gt;
    &lt;/dependencies&gt;

    ...

&lt;/project&gt;
</pre>

<h3>JPA configuration</h3>

The JPA configuration goes here: <code>/src/main/resources/META-INF/persistence.xml</code>. Here's the configuration itself. (Adjust as necessary if you're not using MySQL.)

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;persistence xmlns="http://java.sun.com/xml/ns/persistence"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd"
    version="1.0"&gt;
   
    &lt;persistence-unit name="RunbookManager" transaction-type="RESOURCE_LOCAL"&gt;
        &lt;description&gt;This unit manages runbooks.&lt;/description&gt;
        &lt;provider&gt;org.hibernate.ejb.HibernatePersistence&lt;/provider&gt;
        &lt;jta-data-source&gt;jdbc/RunbookDS&lt;/jta-data-source&gt;
        &lt;properties&gt;
            &lt;property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect" /&gt;
            &lt;property name="hibernate.show_sql" value="false" /&gt;
        &lt;/properties&gt;
    &lt;/persistence-unit&gt;
&lt;/persistence&gt;</pre>

The configuration above is where we declare Hibernate as our JPA provider.

Now let's look at the Spring configuration.

<h3>Spring configuration</h3>

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
        http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa-1.0.xsd
        http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-3.1.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.1.xsd"&gt;
        
    &lt;jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/RunbookDS" resource-ref="true" /&gt;
    
    &lt;bean id="entityManagerFactory"
        class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean"
        p:persistenceUnitName="RunbookManager"
        p:dataSource-ref="dataSource" /&gt;

    &lt;bean id="transactionManager"
        class="org.springframework.orm.jpa.JpaTransactionManager"
        p:entityManagerFactory-ref="entityManagerFactory" /&gt;

    &lt;jpa:repositories base-package="com.example.runbooks.repo" /&gt;
    
    &lt;tx:annotation-driven /&gt;

    ... other beans ...

&lt;/beans&gt;
</pre>

Note the use of the <code>jpa</code> namespace to declare a package containing the repositories. This package contains the various interfaces we're about to define. The <code>&lt;jpa:repositories&gt;</code> configuration tells Spring to scan for interfaces and create the repository implementations, magically.

<h3>Repository interfaces</h3>

OK, now we've made it to the good stuff. We'll look at a few examples here.

To create a new DAO, we simply extend the <code>JpaRepository</code> interface, which is part of Spring Data JPA. It takes two type parameters: the relevant entity type, and its ID type. The interface comes with a bunch of standard CRUD operations and queries; see the <a href="http://static.springsource.org/spring-data/data-jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html">JpaRepository API documentation</a> for details on those.

<h4>Example 1: A barebones repo</h4>

First, the most barebones example would be where we're perfectly happy with the standard CRUD and query operations that the Spring Data <code>JpaRepository</code> already provides. In that case, all we have to do is extend the interface and we're done.

<pre>package com.example.runbooks.repo;

import org.springframework.data.jpa.repository.JpaRepository;
import com.example.runbooks.model.RunbookGroup;

public interface RunbookGroupRepo
    extends JpaRepository&lt;RunbookGroup, Long&gt; { }</pre>

With that simple interface definition, Spring Data JPA will be able to create an implementation dynamically that gives us methods like <code>count()</code>, <code>findAll()</code>, <code>findOne()</code>, <code>save()</code>, <code>delete()</code>, <code>deleteAll()</code>, <code>deleteInBatch()</code> and more for free.

<h4>Example 2. A simple custom query</h4>

Say we have a <code>ChapterType</code> entity with a <code>key</code> property, and we want a query that can find chapter types by key. No problem. We can use conventions around method names to tell Spring Data JPA which query we'd like to see:

<pre>package com.example.runbooks.repo;

import org.springframework.data.jpa.repository.JpaRepository;
import com.example.runbooks.model.ChapterType;

public interface ChapterTypeRepo
    extends JpaRepository&lt;ChapterType, Long&gt; {

    ChapterType findByKey(String key);
}</pre>

Spring Data maps the method to a query that effectively accomplishes

<blockquote>
<code>from ChapterType where key = :key</code>
</blockquote>

<h4>Example 3. A more complex custom query</h4>

The scheme from example 2 above extends to cases where the properties in question are complex, in a couple of different ways: first, they might involve multiple conditions in the "where" clause; second, they might involve joins. Suppose, for instance, that we want to find a chapter having a certain runbook ID and a certain chapter number. Suppose also that the JPQL would be

<blockquote>
<code>from Chapter c where c.runbook.id = :runbookId and<br />
c.chapterType.number = :chapterNumber</code>
</blockquote>

Here's how to build a repo supporting that query:

<pre>package com.example.runbooks.repo;

import org.springframework.data.jpa.repository.JpaRepository;
import com.example.runbooks.model.Chapter;

public interface ChapterRepo
    extends JpaRepository&lt;Chapter, Long&gt; {

    Chapter findByRunbookIdAndChapterTypeNumber(
        Long runbookId, Integer chapterNumber);
}</pre>

The convention implicit in the method name is fairly obvious, especially in light of the JPQL query. The convention can admittedly lead to some awkward method names, as it does in this case. (A better name might be <code>findByRunbookIdAndChapterNumber()</code>.) But the convenience is tough to beat.

<div class="endnote">Interested in learning more about Spring Data JPA? See my post <a href="http://springinpractice.com/2012/05/11/pagination-and-sorting-with-spring-data-jpa/">Pagination and sorting with Spring Data JPA</a>.</div>
