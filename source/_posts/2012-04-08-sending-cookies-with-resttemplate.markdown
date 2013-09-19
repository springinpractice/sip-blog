---
link: http://springinpractice.com/2012/04/08/sending-cookies-with-resttemplate/
layout: post
title: Sending cookies with RestTemplate
date: 2012-04-08 21:06:18
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Quick Tips]
---
Sometimes it is necessary to send cookies along with requests to a RESTful API. One such example is the JIRA 4.3 API, which requires sending the <code>JSESSIONID</code> to JIRA for session management and authentication purposes. REST purists point out that such usages are not properly RESTful (see <a href="http://blog.mikepearce.net/2010/08/24/cookies-and-the-restful-api/" target="_blank">http://blog.mikepearce.net/2010/08/24/cookies-and-the-restful-api/</a> for a good discussion), and indeed the <code>RestTemplate</code> doesn't directly support sending cookies.

But in the real world, we need to make things work, and so in this quick post I'll show how to send cookies with <code>RestTemplate</code>.

The first thing to bear in mind is that we implement cookies as HTTP headers: the service uses a <code>Set-Cookie</code> response header to tell the client to set a cookie, and the client uses the <code>Cookie</code> request header for subsequent requests. And <code>RestTemplate</code> certainly supports setting request headers.

Here's how I'm pulling down an access-controlled RSS feed from JIRA 4.3:

<pre>HttpHeaders requestHeaders = new HttpHeaders();
requestHeaders.add("Cookie", "JSESSIONID=" + session.getValue());
HttpEntity requestEntity = new HttpEntity(null, requestHeaders);
ResponseEntity rssResponse = restTemplate.exchange(
    "https://jira.example.com/sr/jira.issueviews:searchrequest-xml/18107/SearchRequest-18107.xml?tempMax=1000",
    HttpMethod.GET,
    requestEntity,
    Rss.class);
Rss rss = rssResponse.getBody();</pre>

The trick here is to use the <code>RestTemplate</code>'s <code>exchange()</code> method, as this gives us more control over the request, including request headers. We just encode the cookie as a <code>JSESSIONID=[session ID]</code> request header and send it along.

Have fun!

