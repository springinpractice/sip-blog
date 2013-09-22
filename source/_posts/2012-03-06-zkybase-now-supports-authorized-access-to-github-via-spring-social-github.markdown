---
layout: post
title: "Zkybase now supports authorized access to GitHub via Spring Social GitHub"
date: 2012-03-06 12:04:45
comments: true
categories: [Chapter 11 - CMDB, News]
---
This one's another quick <a title="Zkybase" href="https://github.com/williewheeler/zkybase">Zkybase</a> screenshot post. Using <a title="Spring Social GitHub" href="https://github.com/SpringSource/spring-social-github">Spring Social GitHub</a>, it's easy to access public user and repo information via the GitHubTemplate. But if we want to access private information, or write capabilities, we need to use the <a title="Spring Social" href="http://www.springsource.org/spring-social">Spring Social</a> connection framework. This involves adding a Spring Social web controller to the app, giving users a way to log into Zkybase itself (via <a title="Spring Security" href="http://static.springsource.org/spring-security/site/">Spring Security</a>), and then finally giving them a way to connect their Zkybase account to their GitHub account via OAuth 2.

<h3>Screenshots</h3>

Here's what the GitHub section of the account page looks like before you connect to GitHub:

<a href="http://springinpractice.com/wp-content/uploads/2012/03/disconnected.png"><img class="alignnone  wp-image-681" style="border: 1px solid #EEE;" title="disconnected" src="http://springinpractice.com/wp-content/uploads/2012/03/disconnected.png" alt="" width="565" height="96" /></a>

Once you've done that, the display changes to the following:

<a href="http://springinpractice.com/wp-content/uploads/2012/03/connected.png"><img class="alignnone size-medium wp-image-682" style="border: 1px solid #EEE;" title="connected" src="http://springinpractice.com/wp-content/uploads/2012/03/connected-300x195.png" alt="" width="300" height="195" /></a>

And now we can perform various secure operations on our GitHub objects, like viewing repo hooks:

<a href="http://springinpractice.com/wp-content/uploads/2012/03/hooks3.png"><img class="alignnone  wp-image-683" style="border: 1px solid #eeeeee;" title="hooks" src="http://springinpractice.com/wp-content/uploads/2012/03/hooks3.png" alt="" width="619" height="229" /></a>

I plan to use this integration to support autoprovisioning continuous integration servers, among other things.

If you're interested in the code itself, please see my blog post entitled <a href="http://springinpractice.com/2012/03/06/using-spring-social-github-to-access-secured-github-data-code/">Using Spring Social GitHub to access secured GitHub data [code]</a>.
