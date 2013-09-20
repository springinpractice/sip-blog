---
layout: page
title: "Errata"
date: 2013-09-19 20:00
comments: true
sharing: true
footer: true
---
Despite our best efforts, the book will contain errors. We're collecting those up here with the hope that doing so will help clear up any confusion that such errors engender. If you find errors, please feel free to post them as a comment below and I'll incorporate them into the main body of this page.

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
