---
layout: post
title: "Handling JSON error object responses with Spring's RestTemplate"
date: 2013-10-07 01:14
comments: true
categories: [ "Chapter 13 - Integration", "Quick Tips" ]
---
Some web services return JSON error objects when there's a problem. [GitHub's API](http://developer.github.com/v3/#client-errors) is a good case in point, but the approach is not uncommon. Error objects give the API a way to communicate details beyond what the HTTP status codes indicate.

A challenge when using Spring's [RestTemplate](http://docs.spring.io/spring/docs/3.2.x/javadoc-api/org/springframework/web/client/RestTemplate.html) is that there's not an obvious way to handle these. Normally when we use the `RestTemplate`, we indicate a specific type of response payload we expect to see, and so if an error object comes back instead, then it's not clear what to do.

One possible approach is to add error fields to the various resource data transfer objects (DTOs). While this can work, I'm not a big fan of this approach as it fails to separate the resource modeling and error reporting concerns, which I take to be distinct.

So let's look at a different approach&mdash;one that does in fact separate resource modeling from error reporting.

<!-- more -->

The idea
--------

The concept is to read the response body as a string instead of reading it as an object, and then deserialize the body either as the expected response type or else as an error object, depending on whether the status code was in the HTTP 400 or 500 series (client and server errors, respectively).

How to do it
------------

First we need a special `RestTemplate` configuration. By default `RestTemplate` contains a default [ResponseErrorHandler](http://docs.spring.io/spring/docs/3.2.x/javadoc-api/org/springframework/web/client/ResponseErrorHandler.html) implementation called [DefaultResponseErrorHandler](http://docs.spring.io/spring/docs/3.2.x/javadoc-api/org/springframework/web/client/DefaultResponseErrorHandler.html), which throws an exception when there's an HTTP error. This doesn't work for us, because the exception bubbles out of the `RestTemplate` call, thus abandoning the error object we want to read. So we just need to replace it with a custom handler that doesn't throw the exception. Here's one:

    package myapp.client;
    
    import java.io.IOException;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.http.client.ClientHttpResponse;
    import org.springframework.web.client.ResponseErrorHandler;
    import myapp.util.RestUtil;
    
    public class MyResponseErrorHandler implements ResponseErrorHandler {
        private static final Logger log = LoggerFactory.getLogger(MyResponseErrorHandler.class);
            
        @Override
        public void handleError(ClientHttpResponse response) throws IOException {
            log.error("Response error: {} {}", response.getStatusCode(), response.getStatusText());
        }
    
        @Override
        public boolean hasError(ClientHttpResponse response) throws IOException {
            return RestUtil.isError(response.getStatusCode());
        }
    }

In the code above, the `handleError()` method simply logs the error. It doesn't throw an exception, since again we don't want to prevent `RestTemplate` from reading the error object into a string.

Just for completeness, here's `RestUtil`:

    package myapp.util;
    
    import org.springframework.http.HttpStatus;
    
    public class RestUtil {
        
        public static boolean isError(HttpStatus status) {
            HttpStatus.Series series = status.series();
            return (HttpStatus.Series.CLIENT_ERROR.equals(series)
                    || HttpStatus.Series.SERVER_ERROR.equals(series));
        }
    }

Now we need to configure the `RestTemplate` to use our custom `ResponseErrorHandler`:

    <bean class="org.springframework.web.client.RestTemplate">
        <property name="errorHandler">
            <bean class="myapp.client.MyResponseErrorHandler" />
        </property>
    </bean>

We're going to need an object mapper too (I'm assuming [Jackson 2](http://wiki.fasterxml.com/JacksonHome) here, though in principle the same approach should work for JAXB or Jackson 1 as well). So here's that:

    <bean id="objectMapper" class="com.fasterxml.jackson.databind.ObjectMapper" />

Inject the `RestTemplate` and `ObjectMapper` into your client code. Now here's how to use them to deal with error objects:

    @Inject private RestTemplate restTemplate;
    @Inject private ObjectMapper objectMapper;
    
    public DoodadResources getDoodads() {
        HttpHeaders headers = new HttpHeaders();
        headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);
        HttpEntity<String> request = new HttpEntity<String>(headers);
        ResponseEntity<String> response =
                restTemplate.exchange(DOODAD_URL, HttpMethod.GET, request, String.class);
        String responseBody = response.getBody();
        try {
            if (RestUtil.isError(response.getStatusCode())) {
                MyErrorResource error = objectMapper.readValue(responseBody, MyErrorResource.class);
                throw new RestClientException("[" + error.getCode() + "] " + error.getMessage());
            } else {
                DoodadResources doodads = objectMapper.readValue(responseBody, DoodadResources.class);
                return doodads;
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

Notice that the call to `exchange()` specifies that we want to map the response body to a string, which can handle any JSON (or XML) response. Then we use `RestUtil` again to process the response differently depending on the status series. For errors we parse the response body into a `MyErrorResource` and use it to throw an exception with an app-specific error code (not simply an HTTP status code) and a descriptive message. Of course we can do whatever we want with the detailed error information; this is just an example.

If there's no error, then we can parse the response body into the expected response type, if there is one, and return it. Here we return a `DoodadResources`.

Conclusion
----------

Note that since we're capturing the JSON as a string, we're essentially buffering the entire response before parsing it into the actual object (whether expected or error). This may be inappropriate in cases involving large response payloads. Streaming works better there.

Personally I'd like to see a version of `RestTemplate.exchange()` that supports two response types instead of just one: one for the expected response type and one for the error response type. This would avoid the need to handle error objects ourselves, and would also allow the `RestTemplate` to parse the response body (via Jackson) without having to buffer the entire thing first.

Anyway, that's at least one way to do it. If there are better ways I'd very much appreciate hearing about them.
