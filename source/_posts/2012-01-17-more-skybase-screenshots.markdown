---
layout: post
title: "More Skybase screenshots"
date: 2012-01-17 21:30:03
comments: true
categories: [Chapter 11 - CMDB, Demos, News]
---
Hey Internet people, Willie here. It's been a little while since I've posted <a title="Skybase GitHub site" href="https://github.com/williewheeler/skybase">Skybase</a> screenshots, so here's the work in progress. I explain how to implement this stuff (<a title="Spring Data Neo4j" href="http://www.springsource.org/spring-data/neo4j">Spring Data Neo4j</a>, <a title="Spring/GitHub integration" href="https://github.com/SpringSource/spring-social">Spring/GitHub integration</a>, <a title="JavaScript InfoVis Toolkit" href="http://thejit.org/">JavaScript InfoVis Toolkit</a>, etc.) in chapter 11 of my book <a title="Spring in Practice" href="http://manning.com/wheeler/">Spring in Practice</a> (Manning).

<h3>App overview page</h3>

<a href="http://springinpractice.com/wp-content/uploads/2012/01/app_overview.png"><img class="alignnone size-medium wp-image-587" title="app_overview" src="http://springinpractice.com/wp-content/uploads/2012/01/app_overview-300x258.png" alt="" width="300" height="258" /></a>

Applications are a central concept in Skybase. This is how the app overview page looks so far. (It's just a start.) The concept behind it is that if you want a 360-degree view of an app (dev view, test view, release view and ops view), you come to the app details page and then you can start looking at different views.
<h3>App repository commits page</h3>
<a href="http://springinpractice.com/wp-content/uploads/2012/01/commits.png"><img class="alignnone size-medium wp-image-590" title="commits" src="http://springinpractice.com/wp-content/uploads/2012/01/commits-300x280.png" alt="" width="300" height="280" /></a>

Here's a repo commit history for a given app. Right now it's just integrated with GitHub, but the idea is to provide integrations with other providers too (e.g. BitBucket).
<h3>App repository watchers page</h3>
<a href="http://springinpractice.com/wp-content/uploads/2012/01/watchers.png"><img class="alignnone size-medium wp-image-591" title="watchers" src="http://springinpractice.com/wp-content/uploads/2012/01/watchers-300x226.png" alt="" width="300" height="226" /></a>

Again, an app details view. This time it's GitHub watchers for a given app repo.
<h3>Region details page</h3>
<a href="http://springinpractice.com/wp-content/uploads/2012/01/region.png"><img class="alignnone size-medium wp-image-592" title="region" src="http://springinpractice.com/wp-content/uploads/2012/01/region-300x278.png" alt="" width="300" height="278" /></a>

Visualizations are one of the more exciting features that a CMDB can offer. One of the huge advantages of putting your configuration management data in a database is that you can move away from Visio documents and Gliffy diagrams, and move instead toward data modeling and automatically generating views. Here I'm using the <a title="JavaScript InfoVis Toolkit" href="http://thejit.org/">JavaScript InfoVis Toolkit </a>library to generate an interactive graph visualization. It is a great complement to the underlying <a title="Neo4j" href="http://neo4j.org/">Neo4j</a> graph database.
<h3>Automation view</h3>
<a href="http://springinpractice.com/wp-content/uploads/2012/01/xml.png"><img class="alignnone size-full wp-image-601" title="Automation view" src="http://springinpractice.com/wp-content/uploads/2012/01/xml.png" alt="" width="235" height="254" /></a>

This might strike you as a strange screenshot, but in reality it's an example of the most important view--the automation view. The main reason we want to manage configuration data in a database is that we can build web services (Skybase supports both JSON and XML views) that simultaneously drive automation and human-consumable views of the sort that a support team would use. And the data is accurate because it's what brings the environment into being--<a title="Closed loops" href="http://skydingo.com/blog/?p=311">no more chasing your environment around trying to document it.</a>

If you're interested in checking it out or even getting involved in development, see the <a title="Skybase GitHub site" href="https://github.com/williewheeler/skybase">Skybase GitHub site</a>.