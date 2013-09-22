---
layout: post
title: "More Zkybase screenshots"
date: 2012-01-17 21:30:03
comments: true
categories: [Chapter 11 - CMDB, Demos, News]
---
It's been a little while since I've posted <a title="Zkybase GitHub site" href="https://github.com/williewheeler/zkybase">Zkybase</a> screenshots, so here's the work in progress. I explain how to implement this stuff (<a title="Spring Data Neo4j" href="http://www.springsource.org/spring-data/neo4j">Spring Data Neo4j</a>, <a title="Spring/GitHub integration" href="https://github.com/SpringSource/spring-social">Spring/GitHub integration</a>, <a title="JavaScript InfoVis Toolkit" href="http://thejit.org/">JavaScript InfoVis Toolkit</a>, etc.) in chapter 11 of my book <a title="Spring in Practice" href="http://manning.com/wheeler/">Spring in Practice</a> (Manning).

App overview page
-----------------

![App overview](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-17-more-skybase-screenshots/app_overview.png)

Applications are a central concept in Zkybase. This is how the app overview page looks so far. (It's just a start.) The concept behind it is that if you want a 360-degree view of an app (dev view, test view, release view and ops view), you come to the app details page and then you can start looking at different views.

App repository commits page
---------------------------

![Commits](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-17-more-skybase-screenshots/commits.png)

Here's a repo commit history for a given app. Right now it's just integrated with GitHub, but the idea is to provide integrations with other providers too (e.g. BitBucket).

App repository watchers page
----------------------------

![Watchers](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-17-more-skybase-screenshots/watchers.png)

Again, an app details view. This time it's GitHub watchers for a given app repo.

Region details page
-------------------

![Region](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-17-more-skybase-screenshots/region.png)

Visualizations are one of the more exciting features that a CMDB can offer. One of the huge advantages of putting your configuration management data in a database is that you can move away from Visio documents and Gliffy diagrams, and move instead toward data modeling and automatically generating views. Here I'm using the <a title="JavaScript InfoVis Toolkit" href="http://thejit.org/">JavaScript InfoVis Toolkit </a>library to generate an interactive graph visualization. It is a great complement to the underlying <a title="Neo4j" href="http://neo4j.org/">Neo4j</a> graph database.

Automation view
---------------

![XML](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-17-more-skybase-screenshots/xml.png)

This might strike you as a strange screenshot, but in reality it's an example of the most important view--the automation view. The main reason we want to manage configuration data in a database is that we can build web services (Zkybase supports both JSON and XML views) that simultaneously drive automation and human-consumable views of the sort that a support team would use.
