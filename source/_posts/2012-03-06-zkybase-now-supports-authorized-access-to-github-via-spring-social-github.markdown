---
layout: post
title: "Zkybase now supports authorized access to GitHub via Spring Social GitHub"
date: 2012-03-06 12:04:45
comments: true
categories: [Chapter 11 - CMDB, News]
---
This one's another quick [Zkybase](https://github.com/williewheeler/zkybase) screenshot post. Using [Spring Social GitHub](https://github.com/SpringSource/spring-social-github), it's easy to access public user and repo information via the `GitHubTemplate`. But if we want to access private information, or write capabilities, we need to use the [Spring Social](http://projects.spring.io/spring-social/) connection framework. This involves adding a Spring Social web controller to the app, giving users a way to log into Zkybase itself (via [Spring Security](http://projects.spring.io/spring-security/), and then finally giving them a way to connect their Zkybase account to their GitHub account via OAuth 2.

Screenshots
-----------

Here's what the GitHub section of the account page looks like before you connect to GitHub:

![Disconnected screenshot](http://springinpractice.s3.amazonaws.com/blog/images/2012-03-06-zkybase-now-supports-authorized-access-to-github-via-spring-social-github/disconnected.png)

Once you've done that, the display changes to the following:

![Connected screenshot](http://springinpractice.s3.amazonaws.com/blog/images/2012-03-06-zkybase-now-supports-authorized-access-to-github-via-spring-social-github/connected.png)

And now we can perform various secure operations on our GitHub objects, like viewing repo hooks:

![Hooks screenshot](http://springinpractice.s3.amazonaws.com/blog/images/2012-03-06-zkybase-now-supports-authorized-access-to-github-via-spring-social-github/hooks3.png)

I plan to use this integration to support autoprovisioning continuous integration servers, among other things.

If you're interested in the code itself, please see my blog post entitled [Using Spring Social GitHub to access secured GitHub data](http://springinpractice.com/2012/03/06/using-spring-social-github-to-access-secured-github-data-code/)
