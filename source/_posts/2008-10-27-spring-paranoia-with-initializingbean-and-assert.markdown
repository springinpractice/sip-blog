---
layout: post
title: "Spring paranoia with InitializingBean and Assert"
date: 2008-10-27 14:51:40
comments: true
categories: [Chapter 01 - DI, Quick Tips]
---
In this quick post, I'm going to show you how to use the <code>InitializingBean</code> interface and the <code>Assert</code> utility class, both part of the Spring Framework, to express any deep-seated feelings of initialization paranoia you may have.

Let's just jump right in with a code example.

<pre>package com.example.user.service;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.util.Assert;

import com.example.user.dao.UserDao;

public class UserServiceImpl implements UserService, InitializingBean {
    private UserDao userDao;
    private JavaMailSender mailSender;

    public void setUserDao(UserDao userDao) {
        Assert.notNull(userDao, "userDao can't be null"); // 1
        this.userDao = userDao;
    }

    public void setMailSender(JavaMailSender mailSender) {
        Assert.notNull(mailSender, "mailSender can't be null"); // 2
        this.mailSender = mailSender;
    }

    public void afterPropertiesSet() {
        Assert.notNull(userDao, "userDao required"); // 3
        Assert.notNull(mailSender, "mailSender required");
    }

    ...service methods...
}</pre>

In the example above, we have a hypothetical user service with two dependencies: a DAO and a mail sender. The idea here is that we want to do some sanity checks as we initialize our service bean. First, when setting the user DAO, we want to make sure that we don't set it to a null value <span class="cueball">1</span>. We conduct the same test for the mail sender <span class="cueball">2</span>. In each case we use the Spring <code>Assert</code> utility class to perform the test. (Be sure not to confuse this with the <code>assert</code> keyword from the Java language, or the JUnit class of the same name.)

Finally, we implement the single method required by the <code>InitializingBean</code> interface, <code>afterPropertiesSet</code>. Here we check to see that all required properties have actually been set <span class="cueball">3</span>. If they haven't, then Spring complains. Compliant <code>BeanFactory</code> implementations call the <code>afterPropertiesSet</code> method as part of the standard bean lifecycle; see the <a href="http://static.springframework.org/spring/docs/2.5.5/api/org/springframework/beans/factory/BeanFactory.html">BeanFactory Javadocs</a> for more information.

This technique is handy for being sure that your dependency injections actually happen. This is especially useful when using autowiring, where it's easy to miss the fact that some required injection didn't actually occur.

You can use <code>InitializingBean</code> and <code>Assert</code> for any Spring-managed bean, not just service beans. I've just used a service bean here as an example.

Obviously, if you use the <code>InitializingBean</code> interface then you tie your bean APIs to the Spring Framework. You'll have to decide whether that's something you can live with. With <code>Assert</code>, on the other hand, you're only building in an implementation dependency on Spring, which is less objectionable. (It may still be a minor nuisance if you were to decide to move away from Spring, but nothing you couldn't easily handle. I'd probably say the same thing of using <code>InitializingBean</code> in this case, incidentally.)

Anyway, now you know the pros and cons of the approach. Have fun!

<span class="icon stickyNote">Post migrated from my Wheeler Software site.</span>
