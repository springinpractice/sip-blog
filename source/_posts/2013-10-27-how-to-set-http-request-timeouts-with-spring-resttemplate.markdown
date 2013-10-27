---
layout: post
title: "How to set HTTP request timeouts with Spring RestTemplate"
date: 2013-10-27 10:31
comments: true
categories: [ "Chapter 13 - Integration", "Quick Tips" ]
---
In [How to Send an HTTP Header With Every Request With Spring RestTemplate](http://springinpractice.com/2013/10/27/how-to-send-an-http-header-with-every-request-with-spring-resttemplate/), we looked at how to send a given HTTP header automatically with every request. Here we'll do something similar, which is to automatically apply a request timeout to every request.

<!-- more -->

Here's the Java-based configuration for this technique:

    package com.myapp;
    
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.http.client.ClientHttpRequestFactory;
    import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
    import org.springframework.web.client.RestTemplate;
    
    @Configuration
    public class MyAppConfig {
    
        @Bean
        public RestTemplate restTemplate() {
            return new RestTemplate(clientHttpRequestFactory());
        }
    
        private ClientHttpRequestFactory clientHttpRequestFactory() {
            HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
            factory.setReadTimeout(2000);
            factory.setConnectTimeout(2000);
            return factory;
        }
    }

The XML-based configuration is this:

    <bean class="org.springframework.web.client.RestTemplate">
        <constructor-arg>
            <bean class="org.springframework.http.client.HttpComponentsClientHttpRequestFactory"
                p:readTimeout="2000"
                p:connectTimeout="2000" />
        </constructor-arg>
    </bean>
