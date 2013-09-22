---
layout: post
title: "Calling the GitHub API using Spring's RestTemplate"
date: 2012-01-14 15:15:59
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Tutorials]
---
GitHub has an <a title="GitHub API" href="http://developer.github.com/">RESTful JSON API</a> that we can call from our apps. In this post I'll show how I'm grabbing a GitHub repo's watchers and pull them into my own app using Spring's RestTemplate.

Meet the GitHub API
-------------------

First, take a quick look at the <a title="GitHub Repo Watching API" href="http://developer.github.com/v3/repos/watching/">GitHub Repo Watching API</a>. It's pretty simple, and it doesn't require credentials to call. The URL for my <a title="Zkybase GitHub site" href="https://github.com/williewheeler/zkybase">Zkybase</a> project, for example, is <a title="Zkybase watchers" href="https://api.github.com/repos/williewheeler/zkybase/watchers">https://api.github.com/repos/williewheeler/zkybase/watchers</a>.

Go ahead and click the watcher link above. You'll see that the response is a JSON array of watcher objects.

Note that since the GitHub API is HTTPS-based, you will need to download the GitHub API certificate (using, e.g., openssl) and then import it into your Java keystore (using keytool) so your Java programs can call the API without running into certificate errors.

Create a User class for object/JSON mapping
-------------------------------------------

We're going to use <a title="Jackson JSON processor" href="http://jackson.codehaus.org/">Jackson</a> to map the GitHub JSON to Java objects. We'll need a `User` class for that. Here it is:

    package org.skydingo.zkybase.integrations.github.model;
    
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
    }

Nothing too amazing here. Note that I'm using `@JsonProperty` to clarify the mapping from the `avatar_url` and `gravatar_id` JSON fields to the `avatarUrl` and `gravatarId` Java properties, since Jackson doesn't automatically handle underscores or anything like that.

The Maven dependency for the `@JsonProperty` annotation is

    <dependency>
        <groupId>org.codehaus.jackson</groupId>
        <artifactId>jackson-mapper-asl</artifactId>
        <version>1.9.3</version>
    </dependency>

Get the data using RestTemplate
-------------------------------

Now that we have `User`, all we need to do is grab the data using `RestTemplate`. Easy:

    package org.skydingo.zkybase.service;
    
    import javax.inject.Inject;
    import org.skydingo.zkybase.integrations.github.model.User;
    import org.skydingo.zkybase.model.Application;
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
    }

Recall that GitHub returns an array of watchers. That's why we're using `User[].class` in the call. `RestTemplate` substitutes the `user` and `repo` values into the `GITHUB_WATCHER_URI` string for us.

Spring configuration
--------------------

All we need to do is put a `RestTemplate` instance on the app context:

    <bean class="org.springframework.web.client.RestTemplate" />

Success!
--------

After a little hacking at the UI:

![Zkybase SCM page](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-14-calling-the-github-api-using-springs-resttemplate/skybase_scm3.png)

See [Zkybase/GitHub integration](http://springinpractice.com/2012/01/14/skybasegithub-integration/) for more context as to why we'd want to integrate our CMDB with GitHub.
