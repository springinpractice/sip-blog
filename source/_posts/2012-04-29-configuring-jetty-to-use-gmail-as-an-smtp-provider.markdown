---
link: http://springinpractice.com/2012/04/29/configuring-jetty-to-use-gmail-as-an-smtp-provider/
layout: post
title: Configuring Jetty to use Gmail as an SMTP provider
date: 2012-04-29 16:13:34
comments: true
categories: [Chapter 08 - Communicating, Chapter 09 - Comments, Quick Tips]
---
In chapter 8 of <a href="http://www.manning.com/wheeler/">Spring in Practice</a>, recipes 8.2 and 8.3 require a JNDI-exposed JavaMail session backed by an SMTP provider. Here I'll show how to set that up in Jetty 6. For SMTP we'll use Gmail, which provides a free SMTP service to anybody with a Gmail account.

Here's the <code>jetty-env.xml</code> configuration supporting the goals above.

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN" "http://jetty.mortbay.org/configure.dtd"&gt;
&lt;Configure class="org.mortbay.jetty.webapp.WebAppContext"&gt;
    &lt;New id="repository" class="org.mortbay.jetty.plus.naming.Resource"&gt;
        &lt;Arg&gt;mail/Session&lt;/Arg&gt;
        &lt;Arg&gt;
            &lt;New class="org.mortbay.naming.factories.MailSessionReference"&gt;
                &lt;Set name="user"&gt;[your_gmail_username]&lt;/Set&gt;
                &lt;Set name="password"&gt;[your_gmail_password]&lt;/Set&gt;
                &lt;Set name="properties"&gt;
                    &lt;New class="java.util.Properties"&gt;
                        &lt;Put name="mail.user"&gt;[your_gmail_username]&lt;/Put&gt;
                        &lt;Put name="mail.password"&gt;[your_gmail_password]&lt;/Put&gt;
                        &lt;Put name="mail.transport.protocol"&gt;smtp&lt;/Put&gt;
                        &lt;Put name="mail.smtp.host"&gt;smtp.gmail.com&lt;/Put&gt;
                        &lt;Put name="mail.smtp.port"&gt;587&lt;/Put&gt;
                        &lt;Put name="mail.smtp.auth"&gt;true&lt;/Put&gt;
                        &lt;Put name="mail.smtp.starttls.enable"&gt;true&lt;/Put&gt;
                        &lt;Put name="mail.debug"&gt;true&lt;/Put&gt;
                    &lt;/New&gt;
                &lt;/Set&gt;
            &lt;/New&gt;
        &lt;/Arg&gt;
    &lt;/New&gt;

    ... other configuration (e.g. JDBC DataSource) ...
&lt;/Configure&gt;</pre>

This configuration allows us to grab the mail session using the <code>mail/Session</code> name from the Spring configuration file:

<pre>&lt;jee:jndi-lookup id="mailSession" jndi-name="mail/Session" resource-ref="true" /&gt;</pre>

<h3>Alternative configurations</h3>

For information about doing the same thing with Tomcat, or information on configuring your JavaMail session directly into the app (along with the SMTP provider details), see my post <a href="http://springinpractice.com/2008/05/15/send-e-mail-using-spring-and-javamail/">Send e-mail using Spring and JavaMail</a>.

<h3>Problems?</h3>

If you run into an error to the effect that there was a PKIX path building problem, then you need to import the remote certificate into your local truststore. See <a href="http://springinpractice.com/2012/04/29/fixing-pkix-path-building-issues-when-using-javamail-and-smtp/">Fixing PKIX path building issues when using JavaMail and SMTP</a> for details on this issue and how to fix it.
