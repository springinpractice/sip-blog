---
layout: post
title: "Annotation-based transactions in Spring"
date: 2008-03-18 22:52:55
comments: true
categories: [Chapter 02 - Data, Chapter 10 - Integration testing, Tutorials]
---
The post shows how to use transaction management annotations in Spring.  I'm using Spring 2.5 though these features have been available at least since Spring 2.0.

Spring provides several different approaches to transaction management:

<ul class="square">
    <li><b>Programmatic:</b> You can write your transactions using Java source code directly.  Normally you wouldn't do this unless you needed fine-grained control over some particular transaction.</li>
    <li>
        <b>Declarative:</b> You can declare transaction definitions in either a centralized way using XML or in a distributed way using annotations:
        <ul class="square">
            <li>
                <b>XML-based:</b> You can configure transactions in Spring's centralized application context file.  Once again you have options:
                <ul class="square">
                    <li><b>Proxy-based:</b> In this approach you wrap your service beans with transactional proxies.  The proxy and the service bean both implement the same Java interface so the application can't tell them apart.  This is a fairly verbose approach with respect to the actual context configuration though there are techniques you can use to streamline the configuration.</li>
                    <li><b>AOP-based:</b> Here you define AOP aspects to endow your service beans with transactional semantics.  Not quite as verbose as the proxy-based approach.</li>
                </ul>
            </li>
            <li><b>Annotation-based:</b> You can use Java 5 annotations to distribute transaction configuration across Java classes.</li>
        </ul>
    </li>
</ul>

As the title suggests, this article deals with annotation-based configuration.  In my opinion this is the cleanest and most sensible approach: context file configuration is almost nil and the transaction definitions are conveniently located with the service interfaces and/or beans themselves.

Let's start with a basic overview of transactions, and then we'll get to the good stuff.

<h3>Transaction basics</h3>

Feel free to skip this section if you already understand transactions and just want to see how to define transactions in Spring using annotations.

The core idea behind a transaction is that it's a complex operation completed as an all-or-nothing set of simpler operations.  If one of the simpler operations fails, then none of the operations should be applied.

The classic example is making a purchase, and indeed this is probably where the name "transaction" comes from.  There are several steps:

<ol>
<li>Pull funds from buyer's account</li>
<li>Add funds to seller's account</li>
<li>Decrease product inventory</li>
</ol>

A failure could happen at any of those steps.  For example, at step 1, if the buyer doesn't have enough money, then the transaction should fail.  (We wouldn't add funds to the seller's account and we wouldn't decrease product inventory.)  At step 2, if the seller's account has been frozen due to suspected involvement in illegal activities, then the transaction should fail.  At step 3, if the product is out of stock, then again the transaction should fail.

That's just the classic example.  There are other examples that are transactions in the technical sense even though a non-technical person wouldn't normally regard them as transactions <i>per se</i>.  For instance, my web site allows end users to submit comments on articles. I have a database table for comments and a separate link table that links comments to their associated articles.  (I have it this way because I want to be able to relate comments to other entities as well.)  So to add a comment, I need to insert a record into the comment table and then another record into the link table.  If either of those inserts fails, then I don't want to run either one.  That's a transaction too, at least in the narrower technical sense that we're discussing here.

I would say that the all-or-nothing nature of transactions is their defining characteristic, but it is generally recognized that there are four major characteristics we have in mind when we call something a transaction.  They are the ACID properties.

<h4>ACID properties</h4>

ACID is an acronym that stands for atomic, consistent, isolated and durable.  An operation is a transaction if and only if it has these four characteristics, which I'll describe briefly:

<ul class="square">

<li><b>Atomic:</b> This is the aforementioned all-or-nothing property. Even though a transaction comprises multiple operations, it is completed as a single logical unit of work.  Either the whole thing succeeds or none of it does.</li>

<li><b>Consistent:</b> We want our "transactional resource" (which in most cases means our database) to be consistent with reality.  So for example if I have 4,132 widgets in stock after I sell you five of them, then I want the database to say that I have 4,132 widgets in stock after I run a <code>sell(you, 5, widget)</code> transaction. The consistency property means that after a transaction completes, the database (or whatever transactional resource we're using) matches up with the real world.</li>

<li><b>Isolated:</b> In general we want to be able to run lots of transactions against the database at the same time, and we don't want to make a big mess out of things in the process.  The isolation property means that when you run a transaction, it achieves its results without interference from other transactions.  That doesn't mean that the transactions can't run concurrently&mdash;it just means that the transactions shouldn't corrupt each other in the process.  In practice it's nontrivial to strike the right balance between concurrency and isolation; transactions however exhibit some level of isolation, and well-designed transactions exhibit the "right" amount of isolation.</li>

<li><b>Durable:</b> This just means that after a transaction has ended, its results are made persistent.  Probably they would have called this property "persistent" instead of "durable", but "ACIP" is not as cool as "ACID".</li>

</ul>

<h4>Defining individual transactions</h4>

Now that we know what transactions are, we need to understand how individual transactions are actually specified.  Defining a transaction involves picking a chunk of code (maybe a method, maybe a set of methods, maybe a block of code inside a method) and setting values for the following five parameters:

<ul class="square">

<li><b>Propagation behavior:</b> When defining a transaction we must specify its "boundaries": where does the transaction start and where does it end?  In Spring (and also in EJB), the boundaries of a transaction may or may not coincide with the boundaries of a Java method.  In one respect this is not unlike using the <code>synchronized</code> keyword: if one synchronized method calls another on the same class, then the protected scope includes both methods, not just one, and so the boundaries do not coincide with a single method.  But the difference is that with transactions you have a lot more flexibility as far as defining the boundaries.  Certainly it is possible to enter a transaction starting from one method and then include additional method calls within the scope of that initial transaction.  But you can do other things too, such as declare that calls to other methods open up new transactions, or that a given method must always run within the scope of an existing transaction, etc.  I won't go over all the options here but there are seven and you can see them here: <a href="http://static.springframework.org/spring/docs/2.5.0/api/org/springframework/transaction/TransactionDefinition.html">transaction propagation options</a>.  Probably <code>PROPAGATION_REQUIRED</code> is the most typical.</li>

<li><b>Isolation level:</b> I mentioned above that in practice concurrency trades against isolation.  (Once again there's an analogy with threads, incidentally.)  You can specify the level of isolation you need for a given transaction; the general rule of thumb is assign the weakest isolation level you can safely get away with so as to maximize concurrency.  Again I won't go into the details but you can find them here: <a href="http://static.springframework.org/spring/docs/2.5.0/api/org/springframework/transaction/TransactionDefinition.html">transaction isolation levels</a>.  The most typical cases are <code>ISOLATION_READ_COMMITTED</code> and <code>ISOLATION_REPEATABLE_READ</code>.  The <code>ISOLATION_READ_UNCOMMITTED</code> is so weak as to be almost useless in practice, and <code>ISOLATION_SERIALIZABLE</code> is so strong that you don't want to use it unless you really, really need the safety it provides: it's a concurrency-killer.</li>

<li><b>Timeout:</b> I'm currently reading a great book on software development called <a href="http://www.pragprog.com/titles/mnee">Release It!: Design and Deploy Production-Ready Software</a>, by Michael T. Nygard.  In a nutshell this book makes the case that designing software against a functional requirements document isn't (at all) the same thing as designing software for the data center, and to do the latter you need to assume that serious bugs will infiltrate your production releases and design your software to deal with that reality.  One of the key insights is that problems occur primarily at integration points, and one of the strategies for dealing with that is to use timeouts. Transaction timeouts allow the calling application to give up if the database hasn't completed the transaction within a specifiable interval of time.</li>

<li><b>Read-only:</b> This flag indicates whether the transaction will be, well, read-only.  If so, you can flag it as such and the underlying database or other resource can apply optimizations based on that fact.</li>

<li><b>Rollback behavior:</b> By default, both Spring and EJB roll back a transaction (i.e., cancel it without applying any of the changes) when runtime exceptions, but not when checked exceptions occur.  You can modify that behavior.</li>

</ul>

With that overview concluded, let's now examine Spring's annotation-based transaction management.  It is ridiculously straightforward.

<h3>Annotating the Java interface</h3>

With Spring, you typically apply transaction annotations either to Java interfaces for service beans, or else to the service beans themselves.  If you apply the annotations to the interfaces then the transaction definition holds true for any classes implementing those interfaces.  We'll do that here, but remember that you can apply the annotations to the service beans themselves as well.

Here's an interface for a simple article service.  This service allows the application to get an article by ID, and also to post a comment for a given article.

<pre>package myapp.service;

import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import myapp.model.Article;
import myapp.model.Comment;

@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.DEFAULT,
    readOnly = true)
public interface ArticleService {
    
    Article getArticle(long articleId);
    
    @Transactional(readOnly = false)
    void postComment(long articleId, Comment comment);
}</pre>

<div class="alert warning">In this example I'm annotating the <code>ArticleService</code> interface, but that works only with interface-based proxies, not for class-based proxies. In general it's better to place the <code>@Transactional</code> annotation on concrete classes, not interfaces. &mdash; Willie</div>

The first thing to notice is again just how ridiculously simple this is.  I probably don't need to explain it but I will anyway just to be safe.  The <code>@Transactional</code> annotation we define on the interface is setting default values for the various methods that form the interface.  So we are saying here that unless the method itself specifies otherwise, we want <code>Propagation.REQUIRED</code> for the propagation behavior, we want the default isolation level, and we want to flag methods as read-only.

The only other piece to mention is that I've overridden the <code>readOnly</code> setting for the <code>postComment</code> method since that's obviously a write method.

If it could be any simpler I'm not sure how.  Now let's look at the Spring configuration file.

<h3>The Spring configuration file</h3>

Here it is:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd"&gt;
    
    &lt;!-- Wire beans here --&gt;
    ...

    &lt;bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager"&gt;
        ...
    &lt;/bean&gt;
    
    &lt;!-- This tells Spring to activate annotation-driven transactions --&gt;
    &lt;tx:annotation-driven/&gt;
&lt;/beans&gt;</pre>

That's it&mdash;really!  Make sure you include both the <code>aop</code> and <code>tx</code> namespaces and schema locations since that's where the magic lives.  You have to wire your beans up just like you normally would (though note that you can even <a href="spring-autowiring-annotations.html">use annotations to autowire your beans</a>).  And you have to provide a transaction manager of one kind or another.  I'm using the Hibernate 3 <code>HibernateTransactionManager</code> but use whatever you want. If you give your transaction manager the id <code>transactionManager</code>, then you can take advantage of a little Spring convention-over-configuration: <code>&lt;tx:annotation-driven&gt;</code> will pick that up automatically.  (And if you want to call your transaction manager something else, that's fine too.  Just reference that ID using the <code>transaction-manager</code> attribute of the <code>&lt;tx:annotation-driven&gt;</code> element.)

Oh, the <code>&lt;tx:annotation-driven&gt;</code> element just tells Spring that you want it to pick the transaction definitions up from the Java annotations.  I'm not sure how it works behind the scenes, but I think Spring will run through the beans in your context and create the necessary transaction aspects.  Not positive about that&mdash;if you know the details, I'm interested&mdash;but in any case you end up with annotation-driven transactions.  Nice!

<div class="endnote">Post migrated from my Wheeler Software site.</div>
