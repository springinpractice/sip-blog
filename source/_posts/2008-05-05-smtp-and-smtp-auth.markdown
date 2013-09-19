---
link: http://springinpractice.com/2008/05/05/smtp-and-smtp-auth/
layout: post
title: SMTP and SMTP-AUTH
date: 2008-05-05 00:31:39
comments: true
categories: [Chapter 08 - Communicating, Tutorials]
---
This article explains the basics of the Simple Mail Transport Protocol (SMTP) and its extension, SMTP-AUTH. SMTP is the <i>de facto</i> standard application layer protocol for sending e-mail across the Internet, and SMTP-AUTH augments SMTP by supporting client authentication, which allows clients to use an SMTP server as an e-mail relay.

If you haven't played with SMTP over telnet before, it's entertaining and possibly even eye-opening. You'll learn how spammers use SMTP and SMTP-AUTH to achieve their nefarious ends. Hopefully you won't use it for that but this article explains enough of SMTP that you'll understand how to use and abuse it.

<div class="alert warning"><strong>WARNING: OUR TELNET SESSION IS NOT ENCRYPTED.</strong> Even though we use base64 encoding to send the username/password pair to the server, base64 is not encryption. It prevents casual observers from seeing your password, but it can easily be reversed and hence you are basically sending your password in the clear. If you manually start a telnet session such as the one below (and do so only if you are comfortable that you understand the risks involved), I strongly suggest changing your password on the SMTP server immediately afterward.</div>

<h3>Send yourself an e-mail from &lt;insert_sith_lord_here&gt;</h3>

Have you ever wanted to receive an e-mail from your favorite Sith Lord? Let's open up a telnet session that does just that. To do that you will need an SMTP server, which as mentioned above allows you to send e-mail over the Internet. You will also need to know your username and password for the SMTP server as most SMTP servers require that. You can get the SMTP server's host and port from your ISP, and presumably you set the username and password up with your ISP as well.

<pre>$ telnet smtp.example.com 25

S: 220 smtp.example.com ESMTP Sendmail 8.13.8/8.13.6; Thu, 27 Mar 2008 23:14:59 -0700

C: EHLO wheelersoftware.com

S: 250-smtp.example.com Hello wheelersoftware.com [204.13.10.15], pleased to meet you
S: 250-ENHANCEDSTATUSCODES
S: 250-PIPELINING
S: 250-EXPN
S: 250-VERB
S: 250-8BITMIME
S: 250-SIZE 20000000
S: 250-DSN
S: 250-ETRN
S: 250-AUTH LOGIN PLAIN
S: 250-STARTTLS
S: 250-DELIVERBY
S: 250 HELP

C: AUTH LOGIN

S: 334 VXNlcm5hbWU6

C: d2lsbGll

S: 334 UGFzc3dvcmQ6

C: ZnVuc210cA==

S: 235 2.0.0 OK Authenticated

C: MAIL FROM:&lt;darth.vader@deathstar.com&gt;

S: 250 2.1.0 &lt;darth.vader@deathstar.com&gt;... Sender ok

C: RCPT TO:&lt;willie@example.com&gt;

S: 250 2.1.5 &lt;willie@example.com&gt;... Recipient ok

C: DATA

S: 354 Enter mail, end with "." on a line by itself

C: Date: Thu, 27 Mar 2008 23:12:49 -0700 (MST)
C: From: darth.vader@deathstar.com
C: To: willie@example.com
C: Subject: Great article
C: 
C: Hi Willie,
C: I enjoyed your article on TCP/IP-based application protocols.
C: Join me, and together we can rule the galaxy as father and son.
C: Darth Vader
C: .

S: 250 2.0.0 m2S6ExD6029743 Message accepted for delivery

C: QUIT

S: 221 2.0.0 smtp.example.com closing connection

[The server closes the connection]</pre>

<h3>Session analysis</h3>

<pre>$ telnet smtp.example.com 25

S: 220 smtp.example.com ESMTP Sendmail 8.13.8/8.13.6; Thu, 27 Mar 2008 23:14:59 -0700</pre>

The session begins with me telnetting to an SMTP server using port 25, which is the default SMTP port. If you don't know the location of an SMTP server on your network, you'll need to talk to your local sysadmin or else your ISP.

The server issues a 220 reply code (indicating that the service is ready) and identifies itself as being an ESMTP (Extended SMTP) server.

<pre>C: EHLO wheelersoftware.com

S: 250-smtp.example.com Hello wheelersoftware.com [204.13.10.15], pleased to meet you
S: 250-ENHANCEDSTATUSCODES
S: 250-PIPELINING
S: 250-EXPN
S: 250-VERB
S: 250-8BITMIME
S: 250-SIZE 20000000
S: 250-DSN
S: 250-ETRN
S: 250-AUTH LOGIN PLAIN
S: 250-STARTTLS
S: 250-DELIVERBY
S: 250 HELP</pre>

SMTP has a <code>HELO</code> command that one uses to communicate the client host to the SMTP server, but ESMTP servers support <code>EHLO</code> (which stands for "Extended HELLO"), which returns a list of extensions (commands and keywords) that the ESMTP server supports. For example, the <code>STARTTLS</code> command allows the client to start a TLS (Transport Layer Security) session with the server so that the communication will be encrypted.
	
<pre>C: AUTH LOGIN

S: 334 VXNlcm5hbWU6

C: d2lsbGll

S: 334 UGFzc3dvcmQ6

C: ZnVuc210cA==

S: 235 2.0.0 OK Authenticated</pre>

	
This is the SMTP-AUTH part. SMTP-AUTH adds to SMTP support for client authentication, and so now I tell the server that I want to log in. The server responds with the humorous (to me, anyway) response "VXNlcm5hbWU6", which is the base64 encoding of "Username:". (You can encode and decode these using <a href="http://www.motobit.com/util/base64-decoder-encoder.asp">this base64 encoder/decoder</a>; now you can see why I said above that base64 doesn't provide you any real protection.) I provide the base64-encoded version of my username (willie &rarr; d2lsbGll). Then the server asks me for my password; "UGFzc3dvcmQ6" is the base64 encoding of "Password:". So I provide it (funsmtp &rarr; ZnVuc210cA==). Server checks the password and gives me the A-OK.
	
<pre>C: MAIL FROM:&lt;darth.vader@deathstar.com&gt;

S: 250 2.1.0 &lt;darth.vader@deathstar.com&gt;... Sender ok

C: RCPT TO:&lt;willie@example.com&gt;

S: 250 2.1.5 &lt;willie@example.com&gt;... Recipient ok

C: DATA

S: 354 Enter mail, end with "." on a line by itself

C: Date: Thu, 27 Mar 2008 23:12:49 -0700 (MST)
C: From: darth.vader@deathstar.com
C: To: willie@example.com
C: Subject: Great article
C: 
C: Hi Willie,
C: I enjoyed your article on TCP/IP-based application protocols.
C: Join me, and together we can rule the galaxy as father and son.
C: Darth Vader
C: .

S: 250 2.0.0 m2S6ExD6029743 Message accepted for delivery</pre>

Here I'm just entering in the e-mail headers and body itself. When I'm done, I just enter . on a line by itself, and the server accepts the message for delivery.

<pre>C: QUIT

S: 221 2.0.0 smtp.example.com closing connection</pre>

Finally I log out.

<h3>Conclusion</h3>

That's it! Nothing new or earth-shattering here, but it's definitely useful to know how the SMTP protocol works and what you can do with it. Happy e-mailing!

<h3>Resources</h3>
<ul>
<li><a href="http://en.wikipedia.org/wiki/Smtp">SMTP - Wikipedia</a>: Nice overview of SMTP, including a sample communications session over telnet.</li>
<li><a href="http://en.wikipedia.org/wiki/SMTP-AUTH">SMTP-AUTH - Wikipedia</a>: Describes the SMTP-AUTH extension to SMTP, which adds client authentication to the SMTP protocol.</li>
<li><a href="http://cr.yp.to/smtp.html">SMTP: Simple Mail Transfer Protocol</a></li>
<li><a href="http://cr.yp.to/smtp/ehlo.html">The EHLO verb and SMTP extensions</a></li>
<li><a href="http://www.greenend.org.uk/rjk/2000/05/21/smtp-replies.html">SMTP reply codes</a></li>
</ul>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
