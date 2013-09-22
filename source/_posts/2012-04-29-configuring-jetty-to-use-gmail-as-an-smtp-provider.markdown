---
layout: post
title: "Configuring Jetty to use Gmail as an SMTP provider"
date: 2012-04-29 16:13:34
comments: true
categories: [Chapter 08 - Communicating, Chapter 09 - Comments, Quick Tips]
---
In chapter 8 of [Spring in Practice](http://www.manning.com/wheeler/), recipes 8.2 and 8.3 require a JNDI-exposed JavaMail session backed by an SMTP provider. Here I'll show how to set that up in Jetty 6. For SMTP we'll use Gmail, which provides a free SMTP service to anybody with a Gmail account.

<!-- more -->

Here's the `jetty-env.xml` configuration supporting the goals above.

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN" "http://jetty.mortbay.org/configure.dtd">
    <Configure class="org.mortbay.jetty.webapp.WebAppContext">
        <New id="repository" class="org.mortbay.jetty.plus.naming.Resource">
            <Arg>mail/Session</Arg>
            <Arg>
                <New class="org.mortbay.naming.factories.MailSessionReference">
                    <Set name="user">[your_gmail_username]</Set>
                    <Set name="password">[your_gmail_password]</Set>
                    <Set name="properties">
                        <New class="java.util.Properties">
                            <Put name="mail.user">[your_gmail_username]</Put>
                            <Put name="mail.password">[your_gmail_password]</Put>
                            <Put name="mail.transport.protocol">smtp</Put>
                            <Put name="mail.smtp.host">smtp.gmail.com</Put>
                            <Put name="mail.smtp.port">587</Put>
                            <Put name="mail.smtp.auth">true</Put>
                            <Put name="mail.smtp.starttls.enable">true</Put>
                            <Put name="mail.debug">true</Put>
                        </New>
                    </Set>
                </New>
            </Arg>
        </New>
    
        ... other configuration (e.g. JDBC DataSource) ...
    </Configure>

This configuration allows us to grab the mail session using the `mail/Session` name from the Spring configuration file:

    <jee:jndi-lookup id="mailSession" jndi-name="mail/Session" resource-ref="true" />

Alternative configurations
--------------------------

For information about doing the same thing with Tomcat, or information on configuring your JavaMail session directly into the app (along with the SMTP provider details), see my post [Send e-mail using Spring and JavaMail](http://springinpractice.com/2008/05/15/send-e-mail-using-spring-and-javamail/).

Problems?
---------

If you run into an error to the effect that there was a PKIX path building problem, then you need to import the remote certificate into your local truststore. See [Fixing PKIX path building issues when using JavaMail and SMTP](http://springinpractice.com/2012/04/29/fixing-pkix-path-building-issues-when-using-javamail-and-smtp/) for details on this issue and how to fix it.
