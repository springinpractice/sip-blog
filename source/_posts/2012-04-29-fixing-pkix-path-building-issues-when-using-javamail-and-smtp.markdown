---
link: http://springinpractice.com/2012/04/29/fixing-pkix-path-building-issues-when-using-javamail-and-smtp/
layout: post
title: Fixing PKIX path building issues when using JavaMail and SMTP
date: 2012-04-29 13:08:01
comments: true
categories: [Chapter 08 - Communicating, Troubleshooting]
---
I'm writing this post in support of <a href="http://springinpractice.com/category/book/chapter-8/">chapter 8</a> in my book <a href="http://www.manning.com/wheeler/">Spring in Practice</a>, which deals with <a href="http://springinpractice.com/2008/05/15/send-e-mail-using-spring-and-javamail/">Spring/JavaMail integration</a>, since it's not always straightforward to <a href="http://springinpractice.com/2012/04/29/configuring-jetty-to-use-gmail-as-an-smtp-provider/">configure an app to use SMTP</a>.

<h3>The problem</h3>

Suppose that you've configured your JavaMail app to send e-mail via an SMTP server, but you get the following error:

<pre>HTTP ERROR 500

Problem accessing /sip/contact.html. Reason:

    Mail server connection failed; nested exception is javax.mail.MessagingException: Can't send command to SMTP host;
  nested exception is:
	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target. Failed messages: javax.mail.MessagingException: Can't send command to SMTP host;
  nested exception is:
	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

Caused by:

org.springframework.mail.MailSendException: Mail server connection failed; nested exception is javax.mail.MessagingException: Can't send command to SMTP host;
  nested exception is:
	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target. Failed messages: javax.mail.MessagingException: Can't send command to SMTP host;
  nested exception is:
	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target; message exception details (1) are:
Failed message 1:
javax.mail.MessagingException: Can't send command to SMTP host;
  nested exception is:
	javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
	at com.sun.mail.smtp.SMTPTransport.sendCommand(SMTPTransport.java:1420)
	at com.sun.mail.smtp.SMTPTransport.sendCommand(SMTPTransport.java:1408)
	at com.sun.mail.smtp.SMTPTransport.ehlo(SMTPTransport.java:847)
	at com.sun.mail.smtp.SMTPTransport.protocolConnect(SMTPTransport.java:384)
	at javax.mail.Service.connect(Service.java:297)
	at org.springframework.mail.javamail.JavaMailSenderImpl.doSend(JavaMailSenderImpl.java:389)
	at org.springframework.mail.javamail.JavaMailSenderImpl.send(JavaMailSenderImpl.java:340)
	at org.springframework.mail.javamail.JavaMailSenderImpl.send(JavaMailSenderImpl.java:336)

        ... snip ...
</pre>

What's going on, and how do you fix it?

<h3>What's going on</h3>

Your Java runtime doesn't trust the certificate.

Normally Java verifies certificates through the standard <a href="http://en.wikipedia.org/wiki/Chain_of_trust">chain of trust</a> mechanism. But if that chain terminates with a certificate that Java doesn't trust, then Java will complain in the way described above.

<h3>How you fix it</h3>

<h4>Step 1. Decide whether you want to "fix" it at all</h4>

Before you tell Java to trust the cert, make sure that that's actually the right thing to do. I don't have any guidance to offer on that particular point, but if you're trying to connect to a well-known service (say Google) and your Java runtime gives you the PKIX issue, that's a red flag.

<div class="alert warning">If for whatever reason you think Java ought to trust the certificate, then stop&mdash;you're done. Don't import the certificate into your truststore until you figure out why it (or one of the certs in the chain) isn't already there.</div>

<h4>Step 2. Download the certificate from the remote SMTP server</h4>

You can use <code>openssl</code> to get the cert, at least on Unix/Linux and MacOS. I think Cygwin provides <code>openssl</code> on Windows too.

Here's an example for mail.kattare.com, which is the SMTP server I happen to be using. Kattare uses STARTTLS so I'm using the <code>-starttls smtp</code> flag:

<pre>openssl s_client -connect mail.kattare.com:2525 -starttls smtp > kattare-smtp.cer</pre>

The call will look like it's hung for a little while, but it hasn't. You can either wait it out or else just hit Ctrl-C, as the part of the response that we're actually interested in returns immediately.

Now open the file with your favorite text editor and strip out everything other than the certificate itself. Here's what the result looks like for the mail.kattare.com cert:

<pre>-----BEGIN CERTIFICATE-----
MIIDfjCCAuegAwIBAgIDFJoPMA0GCSqGSIb3DQEBBQUAME4xCzAJBgNVBAYTAlVT
MRAwDgYDVQQKEwdFcXVpZmF4MS0wKwYDVQQLEyRFcXVpZmF4IFNlY3VyZSBDZXJ0
aWZpY2F0ZSBBdXRob3JpdHkwHhcNMTAwOTE5MDEzMDUxWhcNMTIxMTIwMTQ0MjI4
WjCB4TEpMCcGA1UEBRMgRFZxc0c5bkIxUGNleTNZVUFzY3otOFNWV0ZnL2Y1aU8x
CzAJBgNVBAYTAlVTMRYwFAYDVQQKDA0qLmthdHRhcmUuY29tMRMwEQYDVQQLEwpH
VDE3NDM3OTM5MTEwLwYDVQQLEyhTZWUgd3d3LnJhcGlkc3NsLmNvbS9yZXNvdXJj
ZXMvY3BzIChjKTEwMS8wLQYDVQQLEyZEb21haW4gQ29udHJvbCBWYWxpZGF0ZWQg
LSBSYXBpZFNTTChSKTEWMBQGA1UEAwwNKi5rYXR0YXJlLmNvbTCBnzANBgkqhkiG
9w0BAQEFAAOBjQAwgYkCgYEA3E9EOUrXoPLgz/N1MFB4xtld2NyDJK5jzPk313VQ
dldvYY8SOd6XqnO/WAmm/2FaFRjhEZ7HcNPAauVeMXW2YQVGSkimeWd8ZDbKU8o6
vJuFJmnRpfxIIZxS1gzJanFrv7v+TtlIQRDP/YI5OnXkZ0sSLVBb2MK7wHLDbtej
dhECAwEAAaOB1TCB0jAfBgNVHSMEGDAWgBRI5mj5K9KylddH2CMgEE8zmJCf1DAO
BgNVHQ8BAf8EBAMCBPAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMCUG
A1UdEQQeMByCDSoua2F0dGFyZS5jb22CC2thdHRhcmUuY29tMDoGA1UdHwQzMDEw
L6AtoCuGKWh0dHA6Ly9jcmwuZ2VvdHJ1c3QuY29tL2NybHMvc2VjdXJlY2EuY3Js
MB0GA1UdDgQWBBRCAb04OmdhLLLRIAtJGNgYnJKQcjANBgkqhkiG9w0BAQUFAAOB
gQCZQVc8nNHGo5Sr1hh9ZMBK2bcivXqLeJkOVt2pQ0OoMWDsq7/ei4njcN5QJXf0
mK3Qb4bUkdJUemS3QITRXVqNnBZaP0XUAKBxK5htwHJLuQ83q71Td6NkqSj4yS35
jM3JXG7LRkr/G6M24RCxBKONckQy+3j1wdy/jZwfisilPg==
-----END CERTIFICATE-----</pre>

<h4>Step 3. Import the certificate into your local truststore</h4>

You'll want to import the cert into the truststore in your Java home directory. My Java home is at

<pre>/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home</pre>

but yours is probably somewhere else. Anyway, once you find it, the truststore is at <code>/lib/security/jssecacerts</code>, at least if you're using JSSE.

<pre>sudo keytool -import -alias [alias] -file [cert_file] -keystore [java_home]/lib/security/jssecacerts</pre>

For example, for me it's:

<pre>sudo keytool -import -alias mail.kattare.com -file kattare-smtp.cer -keystore /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home/lib/security/jssecacerts</pre>

Enter your <code>sudo</code> password and then your keystore password, and then answer yes when it asks you whether to trust the certificate. This will import the cert into the keystore (which doubles as a truststore).

<h4>Step 4. Restart your app and try again</h4>

Hopefully this time it works, or at least gets rid of the PKIX error.

<h3>References</h3>

<ul>
<li><a href="http://serverfault.com/questions/131627/how-to-inspect-remote-smtp-servers-tls-certificate">How to inspect remote SMTP server's TLS certificate? [serverfault]</a>: Explains how to get the remote certificate.</li>
<li><a href="http://stackoverflow.com/questions/373295/digital-certificate-how-to-import-cer-file-in-to-truststore-file-using">Digital Certificate: How to import .cer file in to .truststore file using? [stackoverflow]</a>: Explains how to import the certificate into your truststore.</li>
<li><a href="http://en.wikipedia.org/wiki/STARTTLS">STARTTLS [Wikipedia]</a>: Background material on STARTTLS.</li>
</ul>
