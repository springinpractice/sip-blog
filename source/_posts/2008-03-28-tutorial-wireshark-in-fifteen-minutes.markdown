---
layout: post
title: "Tutorial: Wireshark in fifteen minutes"
date: 2008-03-28 01:15:29
comments: true
categories: [Tutorials]
---
If you've ever wanted the fastest possible introduction to the Wireshark network protocol analyzer, today is your lucky day.  I'm going to show you how to do something useful with Wireshark in fifteen minutes, even if you've never used it before.

This tutorial is a quickstart.  It is not at all comprehensive.  But if you just want to try it out then this is for you.

<b>Step 1.</b> <a href="http://www.wireshark.org/">Download</a> and install Wireshark.  I'm using version 0.99.8 for Windows.  The installation is completely straightforward.

<b>Step 2.</b> Start it up.

<b>Step 3.</b> Think of some network application running on your machine you'd like to investigate.  For this tutorial I'll take a look at an SMTP (e-mail) session initiated by an application I wrote.  I'll pretend I'm troubleshooting the SMTP communication and to do that I want to see what's happening between my app and the SMTP server.  You can pick whatever you like but pick something where you know the IP address of the remote host.  For instance you might pick an HTTP communication between your machine and a remote host.  Anyway write the IP address down.

<b>Step 4.</b> From the Wireshark menubar, choose Capture &rarr; Options.  First, pick the interface (i.e., network interface card, or NIC) you want to investigate.  Don't press the Start button yet.

<b>Step 5.</b> <strong>IMPORTANT: TURN <a href="http://en.wikipedia.org/wiki/Promiscuous_mode">PROMISCUOUS MODE</a> OFF!  IF YOU'RE AT WORK, YOUR NETWORK ADMINISTRATOR MAY SEE YOU RUNNING IN PROMISCUOUS MODE AND SOMEBODY MAY DECIDE TO FIRE YOU FOR THAT.</strong>  Don't risk it, especially not for a tutorial.  Don't press the Start button yet.

<b>Step 6.</b> We need to create a capture filter to prevent Wireshark from capturing all network traffic going through the interface we chose in Step 4.  It is surprising just how much network traffic goes through the interface and we don't want to see all of it.  In the text field next to the "Capture Filter" button, type <code>host &lt;ip_address&gt;</code>, substituting in the IP address you care about for the <code>&lt;ip_address&gt;</code> part.  This will create a filter that passes only that traffic either originating from or going to the specified host.

<b>Step 7.</b> Now you can press Start.  Wireshark is now capturing any data involving the specified IP address, whether as a source or as a destination.

<b>Step 8.</b> If you aren't already doing so, run the application of interest.  In this case I'm going to run the SMTP client that I mentioned in Step 3.  You should see a list of packets appear in the Wireshark window.

<b>Step 9.</b>  Let's take a look at the SMTP session that my app had with the SMTP server.  Go to Analyze &rarr; Follow TCP Stream.  You should see the TCP stream content.  Here's what I saw in mine.  I've suppressed certain parts for security reasons but you get the picture:

<pre>220 [suppressed] ESMTP Sendmail 8.13.8/8.13.6; Fri, 28 Mar 2008 01:15:04 -0700
EHLO [suppressed]
250-[suppressed] Hello [suppressed], pleased to meet you
250-ENHANCEDSTATUSCODES
250-PIPELINING
250-EXPN
250-VERB
250-8BITMIME
250-SIZE 20000000
250-DSN
250-ETRN
250-AUTH LOGIN PLAIN
250-STARTTLS
250-DELIVERBY
250 HELP
AUTH LOGIN
334 VXNlcm5hbWU6
[suppressed]
334 UGFzc3dvcmQ6
[suppressed]
535 5.7.0 authentication failed</pre>

The above is an SMTP-AUTH session that shows a failed authentication.  As you can imagine, this sort of visibility can be extremely useful in troubleshooting networked applications.  For instance, here I can see that my app was able to connect with the SMTP server and send my credentials, but the SMTP server rejected them.

That obviously just scratches the surface with respect to Wireshark's capabilities, but that should be enough to get you started.  Have fun!

<div class="endnote">Post migrated from my Wheeler Software site.</div>
