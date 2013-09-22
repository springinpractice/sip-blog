---
layout: post
title: "Skybase/GitHub integration"
date: 2012-01-14 15:55:43
comments: true
categories: [Chapter 11 - CMDB, News]
---
A major goal for <a title="Zkybase GitHub site" href="https://github.com/williewheeler/zkybase">Zkybase</a> is to tie together the activities of developers, testers, release engineers and operations. A few major benefits are:

* **Tool integration:** We can single source app lists and more across development, build, test, deployment, monitoring, knowledge management and other tools.
* **Process streamlining:** Outputs from the dev process feed into the test and deployment processes, which in turn feed into operational processes.
* **Communications across teams:** All the teams work from the same concepts and data, facilitating communication.

One of the first things I want to tackle in Zkybase, then, is supporting some developer-centric concerns. An important one is source control. Obviously Zkybase isn't itself a source control management system, but it makes sense to pull in useful information from the source code repos just to provide a "single pane of glass" where developers and other users can get a holistic view of what an app is all about.

Since I'm using GitHub, Github integration is a logical starting point for Zkybase/SCM integration. GitHub has a <a title="GitHub REST API" href="http://developer.github.com/v3/">REST API</a> that makes this integration easy. I wrote a <a title="Calling the GitHub API using Spring’s RestTemplate" href="http://springinpractice.com/2012/01/14/calling-the-github-api-using-springs-resttemplate/">post</a> that explains the technical approach. Here's the end result:

![Zkybase SCM page](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-14-skybasegithub-integration/skybase_scm4.png)

I suspect that such integrations will be a large part of the value that Zkybase delivers. They will help realize the process, tool and communications benefits I highlighted above.
