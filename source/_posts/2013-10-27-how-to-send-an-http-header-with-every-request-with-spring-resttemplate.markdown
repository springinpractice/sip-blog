---
layout: post
title: "How to send an HTTP header with every request with Spring RestTemplate"
date: 2013-10-27 08:34
comments: true
categories: [ "Chapter 13 - Integration", "Quick Tips" ]
---
In [Know Which Apps Are Hitting Your Web Service](http://springinpractice.com/2013/10/25/know-which-apps-are-hitting-your-web-service/), I showed how to write a servlet filter that enforces the existence of a special HTTP request header.

From a client perspective, it would be nice to send this header automatically, instead of having to set the header manually with every request. This post shows how we can use Spring's `RestTemplate` and `ClientHttpRequestInterceptor` to accomplish this.

<!-- more -->

First we need to implement a `ClientHttpRequestInterceptor`:

    package com.myapp.connector;
    
    import java.io.IOException;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpRequest;
    import org.springframework.http.client.ClientHttpRequestExecution;
    import org.springframework.http.client.ClientHttpRequestInterceptor;
    import org.springframework.http.client.ClientHttpResponse;
    
    public class XUserAgentInterceptor implements ClientHttpRequestInterceptor {
    
        @Override
        public ClientHttpResponse intercept(
                HttpRequest request, byte[] body, ClientHttpRequestExecution execution)
                throws IOException {
    
            HttpHeaders headers = request.getHeaders();
            headers.add("X-User-Agent", "My App v2.1");
            return execution.execute(request, body);
        }
    }

Now we need to configure our `RestTemplate` to use it. Here I'm using Spring's Java-based configuration, but you can implement the same thing with the XML-based configuration too:

    package com.myapp;
    
    import java.util.Collections;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.http.client.ClientHttpRequestInterceptor;
    import org.springframework.web.client.RestTemplate;
    import com.myapp.connector.XUserAgentInterceptor;
    
    @Configuration
    public class MyAppConfig {
    
        @Bean
        public RestTemplate restTemplate() {
            RestTemplate restTemplate = new RestTemplate(clientHttpRequestFactory());
            restTemplate.setInterceptors(Collections.singletonList(new XUserAgentInterceptor()));
            return restTemplate;
        }
    }

With this configuration, any requests you make through the `RestTemplate` will automatically carry the desired HTTP request header.
