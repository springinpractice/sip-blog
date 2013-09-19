---
layout: post
title: "Using Spring Social GitHub to access secured GitHub data"
date: 2012-03-06 00:40:28
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Tutorials]
---
At my <a title="Skydingo blog" href="http://skydingo.com/blog/">Skydingo blog</a> (a devops blog written in collaboration with Paul Jenson) I just wrote a <a href="http://skydingo.com/blog/2012/03/skybase-now-supports-authorized-access-to-github-via-spring-social-github/" title="Skybase now supports authorized access to GitHub via Spring Social GitHub">quick post</a> with some screenshots of an open source, Spring-based CMDB I'm building called <a title="Skybase GitHub site" href="https://github.com/williewheeler/skybase">Skybase</a>. In the current post I want to show some of the code and configuration that's required to make OAuth2-authorized Spring/GitHub integration work.

[caption id="attachment_672" align="alignnone" width="529" caption="Authorize access to get GitHub repo hooks and other sensitive data"]<a href="http://springinpractice.com/wp-content/uploads/2012/03/hooks2.png"><img src="http://springinpractice.com/wp-content/uploads/2012/03/hooks2.png" alt="" title="hooks" width="529" height="195" class="size-full wp-image-672" /></a>[/caption]

<h3>Spring application configuration</h3>

This goes in the app context. I lifted it more or less as-is from the Spring Social reference documentation, except that I'm using GitHub as the provider instead of Facebook, Twitter or LinkedIn.

<pre>&lt;bean id="connectionFactoryLocator" class="org.springframework.social.connect.support.ConnectionFactoryRegistry"&gt;
    &lt;property name="connectionFactories"&gt;
        &lt;list&gt;
            &lt;bean class="org.springframework.social.github.connect.GitHubConnectionFactory"&gt;
                &lt;constructor-arg value="${gitHub.clientId}" /&gt;
                &lt;constructor-arg value="${gitHub.clientSecret}" /&gt;
            &lt;/bean&gt;
        &lt;/list&gt;
    &lt;/property&gt;
&lt;/bean&gt;

&lt;bean id="textEncryptor" class="org.springframework.security.crypto.encrypt.Encryptors" factory-method="noOpText" /&gt;

&lt;bean id="usersConnectionRepository" class="org.springframework.social.connect.jdbc.JdbcUsersConnectionRepository"&gt;
    &lt;constructor-arg ref="dataSource" /&gt;
    &lt;constructor-arg ref="connectionFactoryLocator" /&gt;
    &lt;constructor-arg ref="textEncryptor" /&gt;
&lt;/bean&gt;

&lt;!-- This requires authentication via Spring Security --&gt;
&lt;bean id="connectionRepository"
    factory-bean="usersConnectionRepository"
    factory-method="createConnectionRepository" 
    scope="request"&gt;
    
    &lt;constructor-arg value="#{request.userPrincipal.name}" /&gt;
    &lt;aop:scoped-proxy proxy-target-class="false" /&gt;
&lt;/bean&gt;</pre>

<h3>Spring web configuration</h3>

<pre>&lt;bean class="org.springframework.social.connect.web.ConnectController" /&gt;</pre>

<h3>Service code for the user account page</h3>

Here's how I'm looking up the GitHub user profile information:

<pre>package org.skydingo.skybase.service.impl;

import javax.inject.Inject;

import org.skydingo.skybase.model.UserAccount;
import org.skydingo.skybase.repository.UserAccountRepository;
import org.skydingo.skybase.service.UserAccountService;
import org.springframework.social.connect.Connection;
import org.springframework.social.connect.ConnectionRepository;
import org.springframework.social.github.api.GitHub;
import org.springframework.social.github.api.GitHubUserProfile;
import org.springframework.social.github.api.impl.GitHubTemplate;
import org.springframework.stereotype.Service;

@Service
public class UserAccountServiceImpl extends AbstractCIService implements UserAccountService {
    @Inject private ConnectionRepository connectionRepo;
    
    public GitHubUserProfile getCurrentUserProfile() {
        if (gitHub().isAuthorized()) {
            return gitHub().userOperations().getUserProfile();
        } else {
            return null;
        }
    }
    
    private GitHub gitHub() {
        Connection conn = connectionRepo.findPrimaryConnection(GitHub.class);
        return (conn != null ? conn.getApi() : new GitHubTemplate());
    }

    ... other methods ...
}</pre>

In the <code>gitHub()</code> method you can see that I return an existing <code>GitHub</code> object if the connection exists; otherwise I just create a new <code>GitHub</code> (in the form of the template, which implements <code>GitHub</code>) so that the app can at least perform non-sensitive read operations.

<h3>Service to get the GitHub service hooks</h3>

This one's pretty similar to the above.

<pre>package org.skydingo.skybase.service.impl;

import java.util.List;
import javax.inject.Inject;
import org.skydingo.skybase.model.Application;
import org.skydingo.skybase.service.ApplicationService;
import org.springframework.social.connect.Connection;
import org.springframework.social.connect.ConnectionRepository;
import org.springframework.social.github.api.GitHub;
import org.springframework.social.github.api.GitHubHook;
import org.springframework.social.github.api.impl.GitHubTemplate;
import org.springframework.stereotype.Service;

@Service
public class ApplicationServiceImpl extends AbstractCIService implements ApplicationService {
    @Inject private ConnectionRepository connectionRepo;
    
    public List findHooks(String user, String repo) {
        return gitHub().repoOperations().getHooks(user, repo);
    }

    private GitHub gitHub() {
        Connection conn = connectionRepo.findPrimaryConnection(GitHub.class);
        return (conn != null ? conn.getApi() : new GitHubTemplate());
    }

    ... various other methods ...
}</pre>

I should probably factor out that common <code>gitHub()</code> method. :-)

<h3>User account JSP</h3>

Here's the JSP code for rendering the connect/disconnect buttons. The CSS classes come from <a title="Twitter Bootstrap" href="http://twitter.github.com/bootstrap/">Bootstrap 2.0</a>, in case you were wondering.

<pre>&lt;%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %&gt;
&lt;%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %&gt;
&lt;%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %&gt;
&lt;%@ taglib prefix="social" uri="http://www.springframework.org/spring-social/social/tags" %&gt;

&lt;c:url var="connectUrl" value="/connect" /&gt;
&lt;c:url var="githubUrl" value="/connect/github" /&gt;

&lt;sec:authentication var="user" property="principal" /&gt;

&lt;h1&gt;Account details&lt;/h1&gt;

&lt;section class="first"&gt;
    &lt;div class="well"&gt;
        &lt;table class="grid"&gt;
            &lt;tr&gt;
                &lt;td&gt;Username:&lt;/td&gt;
                &lt;td&gt;&lt;c:out value="${user.username}" /&gt;&lt;/td&gt;
            &lt;/tr&gt;
        &lt;/table&gt;
    &lt;/div&gt;
&lt;/section&gt;

&lt;section&gt;
    &lt;h2&gt;GitHub&lt;/h2&gt;

    &lt;social:connected provider="github"&gt;
        &lt;p&gt;Your Skybase and GitHub accounts are connected.&lt;/p&gt;
        &lt;div class="well"&gt;
            &lt;table class="grid"&gt;
                &lt;tr&gt;
                    &lt;td&gt;Blog:&lt;/td&gt;
                    &lt;td&gt;&lt;c:out value="${gitHubUserProfile.blog}" default="None" /&gt;&lt;/td&gt;
                &lt;/tr&gt;
                &lt;tr&gt;
                    &lt;td&gt;Location:&lt;/td&gt;
                    &lt;td&gt;&lt;c:out value="${gitHubUserProfile.location}" default="None" /&gt;&lt;/td&gt;
                &lt;/tr&gt;
            &lt;/table&gt;
        &lt;/div&gt;
        &lt;form method="post" action="${githubUrl}"&gt;
            &lt;input type="hidden" name="_method" value="delete" /&gt;
            &lt;input class="btn btn-danger" type="submit" value="Disconnect from GitHub" /&gt;
        &lt;/form&gt;
    &lt;/social:connected&gt;

    &lt;social:notConnected provider="github"&gt;
        &lt;p&gt;Your Skybase and GitHub accounts are not yet connected. Connect them for additional Skybase features.&lt;/p&gt;
        &lt;form method="post" action="${githubUrl}"&gt;
            &lt;input type="hidden" name="scope" value="user, repo, gist" /&gt;
            &lt;input class="btn btn-primary" type="submit" value="Connect to GitHub" /&gt;
        &lt;/form&gt;
    &lt;/social:notConnected&gt;
&lt;/section&gt;</pre>

Note the use of the Spring Social <code>&lt;social:connected&gt;</code> and <code>&lt;social:notConnected&gt;</code> tags to show different content based on whether the user is or isn't connected to the provider in question.

Notice also that the connect button sends a hidden <code>scope</code> parameter to the GitHub URL. This allows us to specify the kinds of operation we want the user to authorize.

Anyway, there you have it. This post was a bit quick, so let me know if I left anything important out and I'm happy to add it. Have fun.