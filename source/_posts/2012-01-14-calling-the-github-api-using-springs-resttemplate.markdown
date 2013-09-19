---
layout: post
title: "Calling the GitHub API using Spring's RestTemplate"
date: 2012-01-14 15:15:59
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Tutorials]
---
GitHub has an <a title="GitHub API" href="http://developer.github.com/">RESTful JSON API</a> that we can call from our apps. In this post I'll show how I'm grabbing a GitHub repo's watchers and pull them into my own app using Spring's RestTemplate.
<h3>Meet the GitHub API</h3>
First, take a quick look at the <a title="GitHub Repo Watching API" href="http://developer.github.com/v3/repos/watching/">GitHub Repo Watching API</a>. It's pretty simple, and it doesn't require credentials to call. The URL for my <a title="Skybase GitHub site" href="https://github.com/williewheeler/skybase">Skybase</a> project, for example, is <a title="Skybase watchers" href="https://api.github.com/repos/williewheeler/skybase/watchers">https://api.github.com/repos/williewheeler/skybase/watchers</a>.

Go ahead and click the watcher link above. You'll see that the response is a JSON array of watcher objects.

Note that since the GitHub API is HTTPS-based, you will need to download the GitHub API certificate (using, e.g., openssl) and then import it into your Java keystore (using keytool) so your Java programs can call the API without running into certificate errors.
<h3>Create a User class for object/JSON mapping</h3>
We're going to use <a title="Jackson JSON processor" href="http://jackson.codehaus.org/">Jackson</a> to map the GitHub JSON to Java objects. We'll need a <code>User</code> class for that. Here it is:
<pre>package org.skydingo.skybase.integrations.github.model;

import org.codehaus.jackson.annotate.JsonProperty;

public class User {
    private Long id;
    private String url, login, avatarUrl, gravatarId;

    public Long getId() { return id; }

    public void setId(Long id) { this.id = id; }

    public String getUrl() { return url; }

    public void setUrl(String url) { this.url = url; }

    public String getLogin() { return login; }

    public void setLogin(String login) { this.login = login; }

    @JsonProperty("avatar_url")
    public String getAvatarUrl() { return avatarUrl; }

    public void setAvatarUrl(String avatarUrl) { this.avatarUrl = avatarUrl; }

    @JsonProperty("gravatar_id")
    public String getGravatarId() { return gravatarId; }

    public void setGravatarId(String gravatarId) { this.gravatarId = gravatarId; }
}</pre>
Nothing too amazing here. Note that I'm using <code>@JsonProperty</code> to clarify the mapping from the <code>avatar_url</code> and <code>gravatar_id</code> JSON fields to the <code>avatarUrl</code> and <code>gravatarId</code> Java properties, since Jackson doesn't automatically handle underscores or anything like that.

The Maven dependency for the <code>@JsonProperty</code> annotation is
<pre>&lt;dependency&gt;
    &lt;groupId&gt;org.codehaus.jackson&lt;/groupId&gt;
    &lt;artifactId&gt;jackson-mapper-asl&lt;/artifactId&gt;
    &lt;version&gt;1.9.3&lt;/version&gt;
&lt;/dependency&gt;</pre>
<h3>Get the data using RestTemplate</h3>
Now that we have <code>User</code>, all we need to do is grab the data using <code>RestTemplate</code>. Easy:
<pre>package org.skydingo.skybase.service;

import javax.inject.Inject;
import org.skydingo.skybase.integrations.github.model.User;
import org.skydingo.skybase.model.Application;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class ApplicationService {
    private static final String GITHUB_WATCHER_URI =
        "https://api.github.com/repos/{user}/{repo}/watchers";

    @Inject private RestTemplate restTemplate;

    public User[] getGithubRepoWatchers(String user, String repo) {
        return restTemplate.getForObject(GITHUB_WATCHER_URI, User[].class, user, repo);
    }
}</pre>
Recall that GitHub returns an array of watchers. That's why we're using <code>User[].class</code> in the call. <code>RestTemplate</code> substitutes the <code>user</code> and <code>repo</code> values into the <code>GITHUB_WATCHER_URI</code> string for us.
<h3>Spring configuration</h3>
All we need to do is put a <code>RestTemplate</code> instance on the app context:

<code>&lt;bean class="org.springframework.web.client.RestTemplate" /&gt;</code>
<h3>Success!</h3>
After a little hacking at the UI:

[caption id="attachment_570" align="alignnone" width="529" caption="Skybase SCM page"]<a href="http://springinpractice.com/wp-content/uploads/2012/01/skybase_scm3.png"><img class="size-full wp-image-570" title="skybase_scm" src="http://springinpractice.com/wp-content/uploads/2012/01/skybase_scm3.png" alt="" width="529" height="296" /></a>[/caption]

[See my post <a href="http://skydingo.com/blog/?p=445">Skybase/GitHub integration</a> on the <a href="http://skydingo.com/blog/">Skydingo blog</a> for more context as to why we'd want to integrate our CMDB with GitHub.]