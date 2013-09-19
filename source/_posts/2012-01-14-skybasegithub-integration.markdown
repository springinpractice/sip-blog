---
link: http://springinpractice.com/2012/01/14/skybasegithub-integration/
layout: post
title: Skybase/GitHub integration
date: 2012-01-14 15:55:43
comments: true
categories: [Chapter 11 - CMDB, News]
---
A major goal for <a title="Skybase GitHub site" href="https://github.com/williewheeler/skybase">Skybase</a> is to tie together the activities of developers, testers, release engineers and operations. A few major benefits are:
<ul>
	<li><strong>Tool integration:</strong> We can single source app lists and more across development, build, test, deployment, monitoring, knowledge management and other tools.</li>
	<li><strong>Process streamlining:</strong> Outputs from the dev process feed into the test and deployment processes, which in turn feed into operational processes</li>
	<li><strong>Communications across teams:</strong> All the teams work from the same concepts and data, facilitating communication.</li>
</ul>
One of the first things I want to tackle in Skybase, then, is supporting some developer-centric concerns. An important one is source control. Obviously Skybase isn't itself a source control management system, but it makes sense to pull in useful information from the source code repos just to provide a "single pane of glass" where developers and other users can get a holistic view of what an app is all about.

Since I'm using GitHub, Github integration is a logical starting point for Skybase/SCM integration. GitHub has a <a title="GitHub REST API" href="http://developer.github.com/v3/">REST API</a> that makes this integration easy. I wrote a <a title="Calling the GitHub API using Spring’s RestTemplate" href="http://springinpractice.com/2012/01/14/calling-the-github-api-using-springs-resttemplate/">blog post on Spring in Practice</a> that explains the technical approach. Here's the end result:

[caption id="attachment_447" align="alignnone" width="560" caption="Skybase SCM page"]<a href="http://springinpractice.com/wp-content/uploads/2012/01/skybase_scm4.png"><img class="size-full wp-image-447 " title="Skybase SCM page" src="http://springinpractice.com/wp-content/uploads/2012/01/skybase_scm4.png" alt="" width="560" /></a>[/caption]

I suspect that such integrations will be a large part of the value that Skybase delivers. They will help realize the process, tool and communications benefits I highlighted above.