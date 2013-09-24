---
layout: page
title: "Spring in Practice - The Book"
date: 2013-09-14 23:23
comments: false
sharing: true
footer: true
---
<img src="http://springinpractice.s3.amazonaws.com/blog/images/sip-cover.png" align="right" />

* [About This Blog](#about-this-blog)
* [Code &amp; Setup](#code-and-setup)
* [Errata](#errata)

<section id="about-this-blog">

About This Blog
===============

This is Willie Wheeler's blog for the Manning title, [Spring in Practice](http://manning.com/wheeler/), by [Willie Wheeler](https://twitter.com/williewheeler) and [Joshua White](https://twitter.com/joshuawhite929). The posts here generally supplement what's in the book.

If you need *support* for something in the book, please post your question to the [book's forum at Manning](http://www.manning-sandbox.com/forum.jspa?forumID=503).

</section>

--------------------------------------------------------------------------------

<section id="code-and-setup">

Code &amp; Setup
================

Getting the source code
-----------------------

The source code for [Spring in Practice](http://manning.com/wheeler/) is freely available from GitHub. You don't have to buy the book to download it, though obviously I hope that you'll find the material sufficiently compelling to consider [buying it](http://manning.com/wheeler/) after all. Each chapter has its own repo, and there's a repo for common dependencies as well:

* [Parent POM and common dependencies](https://github.com/springinpractice/sip-top)
* [Chapter 1: Introducing Spring: the dependency injection container](https://github.com/springinpractice/sip01)
* [Chapter 2: Data persistence, ORM &amp; transactions](https://github.com/springinpractice/sip02)
* [Chapter 3: Building web applications with Spring Web MVC](https://github.com/springinpractice/sip03)
* [Chapter 4: Basic web forms](https://github.com/springinpractice/sip04)
* [Chapter 5: Enhancing Spring MVC applications with Web Flow](https://github.com/springinpractice/sip05)
* [Chapter 6: Authenticating users](https://github.com/springinpractice/sip06)
* [Chapter 7: Authorizing user requests](https://github.com/springinpractice/sip07)
* [Chapter 8: Communicating with users and customers](https://github.com/springinpractice/sip08)
* [Chapter 9: Creating a rich text comment engine](https://github.com/springinpractice/sip09)
* [Chapter 10: Integration testing](https://github.com/springinpractice/sip10)
* [Chapter 11: Building a configuration management database](https://github.com/springinpractice/sip11)
* [Chapter 12: Building an article delivery engine](https://github.com/springinpractice/sip12)
* [Chapter 13: Enterprise integration](https://github.com/springinpractice/sip13)
* [Chapter 14: Creating a Spring-based "site-up" framework](https://github.com/springinpractice/sip14)

Each chapter is organized as an incremental set of recipes around some common theme. Each recipe has its own branch in the chapter's repo. Recipe 9.1, for instance, is branch 01 in the chaper 9 repo. (The links above point at master branches, and as such they reflect the last recipe in each chapter.)

Importing the source code into Eclipse (or Spring Tool Suite)
-------------------------------------------------------------

I created a [YouTube video explaining how to import the sample code](http://www.youtube.com/watch?v=0831ETZinmE).

Building and configuration
--------------------------

The book includes an appendix explaining how the Maven builds and project configurations are organized, but I'll go ahead and describe them here as well since some of the MEAP readers and technical reviewers have already asked for better guidance.

Each recipe externalizes user- and/or environment-specific configuration from the main project code. So in general you will <i>not</i> want to directly edit files in the `/sample_conf` directory. Those are there as documentation rather than configuration. Instead you'll want to do the following.

**Step 1: Book configuration setup (one time only).** Skip this step if you've already done this for some other recipe, as it's a one-time setup for the whole book.

Create a directory somewhere on your filesystem to store your app configuration files for this book. For instance, I created

    /Users/williewheeler/projects/sip/conf

Go into your Maven `[user_directory]/.m2/settings.xml` configuration file (create it if it doesn't already exist&mdash;see the [Maven documentation](http://maven.apache.org/settings.html)) and create a `sip.conf.dir` property pointing to whichever directory you want to use as the top-level configuration directory for the apps in this book. Here's a simple example of an `settings.xml` file:

    <?xml version="1.0" encoding="utf-8"?>
    <settings>
        <profiles>
            <profile>
                <id>default</id>
                <properties>
                    <sip.conf.dir>/Users/williewheeler/projects/sip/conf</sip.conf.dir>
                </properties>
            </profile>
        </profiles>
        <activeProfiles>
            <activeProfile>default</activeProfile>
        </activeProfile>
    </settings>

**Step 2: Chapter configuration setup (per chapter).** Each chapter in the book is an incremental series of recipes, so you will need a chapter-specific folder for each one: `sip01, sip02, sip03, ..., sip14`. Those are the names you need to use; if you use `sip3` instead of `sip03`, it won't work. You can create all of them at the beginning if you like, or you can create them as you need them. Either way, create them inside the general configuration directory from step 1 above:

    /Users/williewheeler/projects/sip/conf/sip01
    /Users/williewheeler/projects/sip/conf/sip02 
    /Users/williewheeler/projects/sip/conf/sip03
    ...
    /Users/williewheeler/projects/sip/conf/sip14

For each chapter, copy the full contents of the chapter's first recipe's `/sample_conf` directory into the chapter-specific configuration directory. Don't copy `/sample_conf` itself&mdash;just its contents. Once again, you can do this all at once if you like, though it's probably easier for you if you just do it as you start each new chapter.

**Step 3: Chapter resource setup (per chapter).** Each chapter uses some set of resources (e.g., database, JavaMail, etc.) to provide backend services to the app. In the case of databases, each chapter that uses one has its own database. For Javamail, it probably makes sense to set up a single SMTP service since there's no reason for each chapter to use a separate one. Anyway, you'll need to set these up.

For the databases, the scripts you need are MySQL 5.1 scripts inside the recipe's `/src/main/sql` directory. (These are usually called something like `schema.sql` and `data.sql`, though the naming isn't 100% consistent across all chapters.) If you need something other than MySQL 5.1, you'll probably have to make minor modifications to the scripts. In most cases we're not doing anything fancy in the database scripts at all.

For SMTP, I recommend using Gmail if you don't already have something available. See my post on [Configuring Jetty to use Gmail as an SMTP provider](http://springinpractice.com/2012/04/29/configuring-jetty-to-use-gmail-as-an-smtp-provider/) for more information.

As noted above, this is a once-per-chapter affair, though sometimes the resources don't appear until later recipes in the chapter.

**Step 4: Chapter configuration (per chapter).** Use the `jetty-env.xml` configuration in your chapter-specific configuration directory to configure JNDI resources. Note that not all chapters have a `jetty-env.xml` configuration file since not all chapters use external resources.

Most chapters also include a `/classes/spring/environment.properties` configuration file. Open it up and make any adjustments that seem sensible given your local environment. The details are chapter- and environment-specific. If the meaning of a property isn't clear from the name, then you'll want to consult the chapter itself to see what the property means and how it is being used.

**Step 5: Running the app (per recipe).** You can run the app using the Maven Jetty plugin as follows:

    mvn -e clean jetty:run

When the app starts up, in most cases you will direct your browser to

    http://localhost:8080/sip/

to see the app. There are some exceptions, but the book calls them out where they exist.

</section>

--------------------------------------------------------------------------------

<section id="errata">

Errata
======

Despite our best efforts, the book contains errors. We're collecting those up here with the hope that doing so will help clear up any confusion that such errors engender. Please report errors to [the book's forum at the Manning site](http://www.manning-sandbox.com/forum.jspa?forumID=503).

General
-------

Nothing yet.

Chapter 1
---------

Nothing yet.

Chapter 2
---------

In listing 2.14, the `entityManagerFactory` definition should have a JPA vendor adapter:

    <bean id="entityManagerFactory" ...>
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter" />
        </property>
        ...
    </bean>

Nothing yet.

Chapter 3
---------

Section 3.5.6 incorrectly says that we will discuss `MultipartResolver` in chapter 11 (purportedly about product catalogs, but in fact about CMDBs), and also says that we'll discuss `LocaleResolver` and `ThemeResolver` in chapter 7 (claimed to be about user interfaces; actually about ACLs).

Chapter 4
---------

Nothing yet.

Chapter 5
---------

Nothing yet.

Chapter 6
---------

Nothing yet.

Chapter 7
---------

Nothing yet.

Chapter 8
---------

Nothing yet.

Chapter 9
---------

Nothing yet.

Chapter 10
----------

Nothing yet.

Chapter 11
----------

Nothing yet.

Chapter 12
----------

The summary introduces Chapter 13 as being about product catalogs, but Chapter 13 is really about enterprise integration.

Chapter 13
----------

`/sip13/portal/sample_conf/classes/spring/portal.properties` is missing from the code distribution. You can get it at [the GitHub site](https://github.com/springinpractice/sip13/tree/02/portal/sample_conf/classes/spring).

Chapter 14
----------

Nothing yet.

</section>
