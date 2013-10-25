---
layout: post
title: "Know which apps are hitting your web service"
date: 2013-10-25 00:42
comments: true
categories: [ "Chapter 13 - Integration", "Quick Tips" ]
---
When creating a web service, it's often useful to know which apps are hitting it. I don't mean which users, but instead which apps. The reason is that you may want to coordinate with some other team on something. For example, maybe the team is being too aggressive about request retries, or maybe you want to alert the team to a change in the API. Whatever the reason, it's good to know which apps are calling your service.

HTTP has a `User-Agent` header that can help here. One possible approach is to make that a required header. Unfortunately, this approach isn't ideal. The problem is that HTTP client libraries usually set that header automatically. So if the client application forgets to set the header explicitly, you end up with user agents like `Apache-HttpClient/release (java 1.5)`, which isn't much help at all.

An approach I like better is to define a custom header and make it required. I use `X-User-Agent`, since it really is a user agent we're talking about here.

Here's how to implement this with a servlet filter. No Spring involved here at all; it's just servlet stuff.

    package com.myapp.web.filter;
     
    import java.io.IOException;
    import javax.servlet.Filter;
    import javax.servlet.FilterChain;
    import javax.servlet.FilterConfig;
    import javax.servlet.ServletException;
    import javax.servlet.ServletRequest;
    import javax.servlet.ServletResponse;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
     
    public class XUserAgentFilter implements Filter {
        private static final String X_USER_AGENT = "X-User-Agent";
        
        private String errorJson;
        
        public XUserAgentFilter() {
            String message = 
                "HTTP header '" + X_USER_AGENT + "' is required. Please set it to your application name so we know " +
                "who to contact if there's an issue.";
            this.errorJson = "{ " + wrap("message") + " : " + wrap(message) + " }";
        }
    
        private String wrap(String s) { return "\"" + s + "\""; }
          
        @Override
        public void init(FilterConfig config) throws ServletException { }
     
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
                throws IOException, ServletException {
                 
            HttpServletRequest httpRequest = (HttpServletRequest) request;
            HttpServletResponse httpResponse = (HttpServletResponse) response;
                 
            if (httpRequest.getHeader(X_USER_AGENT) == null) {
                httpResponse.setStatus(422);
                httpResponse.getWriter().println(errorJson);
            } else {
                chain.doFilter(request, response);
            }
        }
    
        @Override
        public void destroy() { }
    }

Of course, you need to configure this filter and a filter mapping in your `web.xml` file.

In the next post I'll show you how to set up your Spring `RestTemplate` to send the `X-User-Agent` header with each request automatically.
