---
layout: post
title: "Spring Social GitHub: revisiting GitHub integration"
date: 2012-01-17 23:44:56
comments: true
categories: [Chapter 11 - CMDB, Chapter 13 - Integration, Tutorials]
---
In my last post, <a title="Calling the GitHub API using Spring’s RestTemplate" href="http://springinpractice.com/2012/01/14/calling-the-github-api-using-springs-resttemplate/">Calling the GitHub API using Spring's RestTemplate</a>, I explained how to call a public endpoint that retrieves a repository's watchers on the <a href="http://developer.github.com/v3/">GitHub API</a> using Spring's RestTemplate.

I was happy to receive a <a href="http://springinpractice.com/2012/01/14/calling-the-github-api-using-springs-resttemplate/#comment-230">comment</a> by <a title="Craig Walls' Twitter page" href="https://twitter.com/#!/habuma">Craig Walls</a> pointing out that <a title="Spring Social" href="http://www.springsource.org/spring-social">Spring Social</a> has a <a title="Spring Social GitHub's GitHub site" href="https://github.com/SpringSource/spring-social-github">GitHub binding</a>, which came with an invitation to contribute code to that binding. Sounded great--I forked the project on GitHub and dug in. I discovered two factoids:
<ol>
	<li>The project is <a title="Gradle" href="http://gradle.org/">Gradle</a>-based. Excellent, because I'd been wanting to try it out for a couple of years since Craig first recommended it to me in at Spring One 2GX. I just never got around to it.</li>
	<li>The Spring Social GitHub binding is nascent. (Understandably, Facebook, Twitter and LinkedIn get the lion's share of development attention.) Outstanding, because I could implement whatever I wanted.</li>
</ol>
I ended up writing a handful of public endpoints for Spring Social GitHub. I focused on the public ones instead of the personal ones requiring OAuth (GitHub supports OAuth2 among others) since the <a title="Skybase GitHub site" href="https://github.com/williewheeler/skybase">Skybase app</a> I'm building probably doesn't need any of the personal endpoints. I'll show you some of what I did since it's fairly straightforward and since some of you might be interested in contributing further code. The Spring Social documentation has a <a title="Contributing code" href="http://static.springsource.org/spring-social/docs/1.0.x/reference/html/implementing.html">chapter on how to create or elaborate a binding</a>, and there are existing bindings for <a href="https://github.com/SpringSource/spring-social-facebook">Facebook</a>, <a href="https://github.com/SpringSource/spring-social-twitter">Twitter</a> and <a href="https://github.com/SpringSource/spring-social-linkedin">LinkedIn</a> that provide good guidance as to how to structure things. The GitHub site has <a title="Mechanics" href="https://github.com/SpringSource/spring-social/wiki/Contributing">useful information on the mechanics as well.</a> I was a little bit lazy and just shot Craig a few questions. He was <a href="https://github.com/SpringSource/spring-social-github/pull/1">good-natured</a> about it, and pointed me to the right examples.
<h3>Implementing support for repository watching in Spring Social GitHub</h3>
<strong>Implementing a user class.</strong> In the last post I implemented a <code>User</code> class and then just called the GitHub URL with the RestTemplate, deserializing the JSON into my User. As it turns out, this isn't that different than what I added to Spring Social GitHub. In fact, the revised version is almost exactly the same:
<pre>package org.springframework.social.github.api;

import java.io.Serializable;
import org.codehaus.jackson.annotate.JsonIgnoreProperties;
import org.codehaus.jackson.annotate.JsonProperty;

@SuppressWarnings("serial")
@JsonIgnoreProperties(ignoreUnknown = true)
public class GitHubUser implements Serializable {
    private Long id;
    private String url;
    private String login;
    private String avatarUrl;
    private String gravatarId;

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
I added the <code>@JsonIgnoreProperties</code> annotation to try to future-proof the class a little, and I renamed it to <code>GitHubUser</code> since that was more in line with the Spring Social convention around naming data transfer objects.

<strong>Creating the client interfaces.</strong> Spring Social basically wraps <code>RestTemplate</code> with provider-specific methods and support for OAuth authorization. So instead of passing <code>https://api.github.com/repos/williewheeler/skybase to RestTemplate.getForObject()</code>, we call a <code>getWatchers()</code> method on the Spring Social GitHub API.

The general pattern is for there to be a top-level Java client interface, which is called <code>Facebook</code>, or <code>Twitter</code>, or <code>LinkedIn</code>, or <code>GitHub</code>, and then for this interface to have smaller sub-APIs to group related operations. This avoids having a monster interface with dozens and dozens of methods. Here, for example, is what the <code>GitHub</code> interface looks like so far:
<pre>package org.springframework.social.github.api;

import org.springframework.social.ApiBinding;
import org.springframework.social.github.api.impl.GitHubTemplate;

public interface GitHub extends ApiBinding {

    RepoOperations repoOperations();

    UserOperations userOperations();
}</pre>
GitHub has a lot more sets of operations than that (e.g. gists, issues, orgs, pull requests, etc.), but I was just getting my feet wet and so I only created a couple of <code>XxxOperations</code> interfaces here.

Here's what <code>RepoOperations</code> looks like so far:
<pre>package org.springframework.social.github.api;

import java.util.List;

public interface RepoOperations {
    List getCollaborators(String user, String repo);
    List getCommits(String user, String repo);
    List getWatchers(String user, String repo);
}</pre>
(As I mentioned, I implemented a few things beyond <code>getWatchers()</code>.)

<strong>Filling in the client implementations.</strong> With the interfaces in place, it was time to write their implementations. This looked pretty close to the code from my previous article, but a little more industrial-strength to handle API modularity and also personal endpoints. Here for instance is the <code>RepoTemplate</code>.
<pre>package org.springframework.social.github.api.impl;

import static java.util.Arrays.asList;

import java.util.List;
import org.springframework.social.github.api.GitHubCommit;
import org.springframework.social.github.api.GitHubUser;
import org.springframework.social.github.api.RepoOperations;
import org.springframework.web.client.RestTemplate;

public class RepoTemplate extends AbstractGitHubOperations
    implements RepoOperations {

    private final RestTemplate restTemplate;

    public RepoTemplate(RestTemplate restTemplate, boolean isAuthorizedForUser) {
        super(isAuthorizedForUser);
        this.restTemplate = restTemplate;
    }

    public List getCollaborators(String user, String repo) {
        return asList(restTemplate.getForObject(
            buildRepoUri("/collaborators"), GitHubUser[].class, user, repo));
    }

    public List getCommits(String user, String repo) {
        return asList(restTemplate.getForObject(
            buildRepoUri("/commits"), GitHubCommit[].class, user, repo));
    }

    public List getWatchers(String user, String repo) {
        return asList(restTemplate.getForObject(
            buildRepoUri("/watchers"), GitHubUser[].class, user, repo));
    }

    private String buildRepoUri(String path) {
        return buildUri("repos/{user}/{repo}" + path);
    }
}</pre>
<strong>Test cases.</strong> I wrote some test cases as well, using the test framework that the Spring guys put together. It is basically a mock server that serves up JSON files so you don't have to hit the real GitHub and put your rate limit in danger.
<h3>Using Spring Social GitHub</h3>
With all that in place, I was excited to try out my new code. Spring Social GitHub hasn't had an actual release yet (Spring Social Core has, but not the GitHub binding), so you'll need to hit a snapshot repository to get it:
<pre>&lt;repository&gt;
    &lt;id&gt;spring-snapshot&lt;/id&gt;
    &lt;name&gt;Spring Maven Snapshot Repository&lt;/name&gt;
    &lt;url&gt;http://maven.springframework.org/snapshot&lt;/url&gt;
&lt;/repository&gt;

...

&lt;dependency&gt;
    &lt;groupId&gt;org.springframework.social&lt;/groupId&gt;
    &lt;artifactId&gt;spring-social-core&lt;/artifactId&gt;
    &lt;version&gt;1.0.1.RELEASE&lt;/version&gt;
&lt;/dependency&gt;
&lt;dependency&gt;
    &lt;groupId&gt;org.springframework.social&lt;/groupId&gt;
    &lt;artifactId&gt;spring-social-github&lt;/artifactId&gt;
    &lt;version&gt;1.0.0.BUILD-SNAPSHOT&lt;/version&gt;
&lt;/dependency&gt;</pre>
Here's the actual Java code:
<pre>import org.springframework.social.github.api.GitHub;
import org.springframework.social.github.api.GitHubUser;

...

GitHub gitHub = ... injected or whatever ...
List&lt;GitHubUser&gt; watchers =
    gitHub.repoOperations().getWatchers("williewheeler", "skybase");</pre>
Couldn't be much easier.

Here are some Skybase screenshots that show what sort of information I'm pulling down from GitHub using Spring Social GitHub:

[gallery columns="2"]