---
layout: post
title: "Send e-mail using Spring and JavaMail"
date: 2008-05-15 21:18:08
comments: true
categories: [Chapter 08 - Communicating, Tutorials]
---
This brief tutorial will show how to send e-mail using Spring and JavaMail. JavaMail can handle e-mail storage as well, but here we're just worrying about sending e-mail.

I happen to be using Spring 2.5 but this ought to work for earlier versions of Spring as well (at least Spring 2.0 I think).

Let's jump right in. You can configure your JavaMail session either in Spring itself or with JNDI. We'll look at both alternatives.

<h3>Alternative 1: Configuring JavaMail with Spring</h3>

You may be operating in an environment where you don't have a JNDI enterprise naming context (ENC) available. No problem; you can just configure JavaMail in the Spring application context configuration as shown below.

<pre>&lt;!-- Mail service --&gt;
&lt;bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl"&gt;
    &lt;property name="host" value="your.smtphost.com"/&gt;
    &lt;property name="port" value="25"/&gt;
    &lt;property name="username" value="yourusername"/&gt;
    &lt;property name="password" value="yourpassword"/&gt;
    &lt;property name="javaMailProperties"&gt;
        &lt;props&gt;
            &lt;!-- Use SMTP-AUTH to authenticate to SMTP server --&gt;
            &lt;prop key="mail.smtp.auth"&gt;true&lt;/prop&gt;
            &lt;!-- Use TLS to encrypt communication with SMTP server --&gt;
            &lt;prop key="mail.smtp.starttls.enable"&gt;true&lt;/prop&gt;
        &lt;/props&gt;
    &lt;/property&gt;
&lt;/bean&gt;</pre>

This bean is, as its name suggests, a mail sender. It is basically a wrapper around JavaMail SMTP, and the configuration reflects that. In the example I'm showing how you would enable SMTP-AUTH (supports authentication to the SMTP server) and TLS (supports message encryption), assuming your SMTP server has those capabilities. For more information see my article <a href="http://springinpractice.com/2008/05/05/smtp-and-smtp-auth/">SMTP and SMTP-AUTH</a>.

<strong>IMPORTANT:</strong> You will need to inject <code>mailSender</code> into your mail-sending service bean.

So that's how to configure JavaMail from Spring. Now here's how to do the same thing with JNDI, which you may want to do if you're running in an environment with a JNDI ENC (like an app server or a servlet container).

<h3>Alternative 2: Configuring JavaMail with JNDI</h3>

<h4>Server JNDI configuration (using Tomcat 6 as an example)</h4>

(For a Jetty configuration using Gmail, see <a href="http://springinpractice.com/2012/04/29/configuring-jetty-to-use-gmail-as-an-smtp-provider/">Configuring Jetty to use Gmail as an SMTP provider</a>.)

First you will need to expose a JavaMail session factory through JNDI in your server environment. This is environment-dependent, but let's look at an example.

Say you're using Tomcat 6. There are a couple things you must do. First, move <code>mail.jar</code> and <code>activation.jar</code> to your <code>tomcat/lib</code> directory. I say "move" rather than "copy" because you will get an odd error if you leave the two JARs in your application classpath. The error is

<pre>java.lang.IllegalArgumentException: 
    Cannot convert value of type [javax.mail.Session] to required type
    [javax.mail.Session] for property 'session': no matching editors
    or conversion strategy found</pre>

Second, define your Tomcat JNDI configuration, which might look like this (e.g. in your <code>context.xml</code> file):

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;Context path="/myapp" docBase="myapp" debug="5" crossContext="false"&gt;

    &lt;!-- JavaMail session factory --&gt;
    &lt;Resource name="mail/Session"
              auth="Container"
              type="javax.mail.Session"
              username="yourusername"
              password="yourpassword"
              mail.debug="true"
              mail.user="yourusername"
              mail.password="yourpassword"
              mail.transport.protocol="smtp"
              mail.smtp.host="your.smtphost.com"
              mail.smtp.auth="true"
              mail.smtp.port="25"
              mail.smtp.starttls.enable="true"/&gt;
&lt;/Context&gt;</pre>

As with the non-JNDI example, I'm configuring for SMTP-AUTH and TLS. If you are using SMTP-AUTH (authenticated SMTP sessions, which you activate using <code>mail.smtp.auth="true"</code>), then you will need to specify the username and password twice, as shown above. Also, if your SMTP server supports it, you can tell JavaMail to encrypt sessions using TLS by setting <code>mail.smtp.starttls.enable=true</code>.

The above discussion applies only to Tomcat 6 (see <a href="http://tomcat.apache.org/tomcat-6.0-doc/jndi-resources-howto.html">Apache Tomcat 6.0 JNDI Resources HOWTO</a> for detailed instructions); you'll need to consult your server docs to expose a JavaMail session factory through JNDI in your environment.

<h4>Spring configuration</h4>

We still need to create a mail sender, but the configuration is simpler since we did all the heavy lifting in <code>context.xml</code> (or whatever, depending on your server environment). So for the Spring application context, all we need is:

<pre>&lt;bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl"&gt;
    &lt;property name="session" ref="mailSession"/&gt;
&lt;/bean&gt;</pre>

<strong>IMPORTANT:</strong> As before, you will need to inject <code>mailSender</code> into your mail-sending service bean.

So that takes care of configuring the mail sender, and also the JavaMail session factory if you are using JNDI. Let's visit one more topic before we dive into the code itself.

<h3>Creating an e-mail template (optional)</h3>

Sometimes the e-mail you want to send fits inside a standard template (e.g., maybe it always has the same sender, or maybe the same recipient, or whatever). You can define a template using Spring. Here's an example:

<pre>&lt;!-- Mail message --&gt;
&lt;bean id="mailMessage" class="org.springframework.mail.SimpleMailMessage"&gt;
    &lt;property name="from"&gt;
        &lt;value&gt;&lt;![CDATA[Simple Application Monitor &lt;noreply@somehost.com&gt;]]&gt;&lt;/value&gt;
    &lt;/property&gt;
    &lt;property name="to"&gt;
        &lt;value&gt;&lt;![CDATA[System Administrator &lt;sysadmin@somehost.com&gt;]]&gt;&lt;/value&gt;
    &lt;/property&gt;
    &lt;property name="subject" value="SAM Alert"/&gt;
&lt;/bean&gt;</pre>

You can include as many e-mail templates in a Spring app context configuration, including none if you don't need a template. Here I've defined a template that specifies a "from" field, a "to" field and a "subject" field. It doesn't specify the date or the body. That template works say for an application monitoring system but it wouldn't work for an e-commerce site's order confirmation e-mail,which would need to have a variable "to" field.

<strong>IMPORTANT:</strong> I didn't show it above, but you will need to inject any e-mail templates (i.e. mail messages) you create into your mail-sending service bean. Otherwise the service bean has no way to use the template.

So that's that. Time for the Java code that actually sends the e-mail.

<h3>How to send the e-mail from your service bean</h3>

It turns out that the coding part is much simpler than the configuration (not that the config was too bad). Let's suppose for the sake of example that we want to send an e-mail based on the template that we defined above, and that you've injected that template into your service bean as <code>mailMessage</code>. Then here's the service bean code that allows you to use the template to send an e-mail:

<pre>SimpleMailMessage message = new SimpleMailMessage(mailMessage);
message.setSentDate(new Date());
message.setText("Blah blah blah...");
mailSender.send(message);</pre>

You can see we're creating a new message based on the <code>mailMessage</code> e-mail template and <code>mailSender</code> that we injected into said service bean.

And that's it!

<span class="icon stickyNote">Post migrated from my Wheeler Software site.</span>
