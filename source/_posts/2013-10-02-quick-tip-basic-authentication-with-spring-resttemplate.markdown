---
layout: post
title: "Quick tip: Basic authentication with Spring RestTemplate"
date: 2013-10-02 02:10
comments: true
categories: [ "Chapter 06 - Authentication", "Chapter 13 - Integration", "Quick Tips" ]
---
Here's a quick tip for using Spring's [RestTemplate](http://docs.spring.io/spring/docs/3.2.x/javadoc-api/org/springframework/web/client/RestTemplate.html) to authenticate to a RESTful web service that uses [HTTP basic authentication](http://en.wikipedia.org/wiki/Basic_access_authentication).

<!-- more -->

There are a couple of things we have to do:

* First we construct an `Authorization` request header that contains (among other things) the user's base 64-encoded credentials.
* Then we invoke the `RestTemplate` in such a way as to send that request header to the service.

Let's start with the `Authorization` header.

Authorization header
--------------------

For the sake of example, suppose that the username is `willie` and the password is `p@ssword`.

The first step is to base 64 encode the string `willie:p@ssword`. In general we want to do this programmatically. The [Apache Commons Codec library](http://commons.apache.org/proper/commons-codec/) is useful for doing this:

    import org.apache.commons.codec.binary.Base64;
    
    ...
    
    String plainCreds = "willie:p@ssword";
    byte[] plainCredsBytes = plainCreds.getBytes();
    byte[] base64CredsBytes = Base64.encodeBase64(plainCredsBytes);
    String base64Creds = new String(base64CredsBytes);

Now let's construct our HTTP request headers, including the `Authorization` header:

    import org.springframework.http.HttpHeaders;
    
    ...
    
    HttpHeaders headers = new HttpHeaders();
    headers.add("Authorization", "Basic " + base64Creds);

Next, let's use the `RestTemplate` to issue the request.

Using RestTemplate to send the request
--------------------------------------

The key here is to use one of the `RestTemplate`'s `exchange()` methods to exchange a request entity for a response entity. Let's imagine that we're going to get account information..

    import org.springframework.http.HttpEntity;
    import org.springframework.http.ResponseEntity;
    
    ...
    
    HttpEntity<String> request = new HttpEntity<String>(headers);
    ResponseEntity<Account> response = restTemplate.exchange(url, HttpMethod.POST, request, Account.class);
    Account account = response.getBody();

In both the request and response, the type parameters represent the body. In the request we aren't sending anything in the body, so we just use `String` as a default, and pass the headers along by themselves. In the response we expect an account, so that's why we have `Account` and `Account.class` there.

Happy basic authenticating.
