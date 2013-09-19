---
layout: post
title: "Mavenizing an empty GitHub project in Eclipse"
date: 2012-05-06 19:30:15
comments: true
categories: [Quick Tips]
---
This post is for myself more than anything else, just because I keep forgetting the steps involved.

I'm using SpringSource Tool Suite 2.9.1.RELEASE, which is based on Eclipse 3.7.2 (Indigo). I have the egit and m2e Eclipse plugins installed.

<h3>The scenario</h3>

You have a brand new, pretty-much-empty GitHub project (other than the README, say&mdash;but no Maven stuff yet), and you want to import it into Eclipse as a Maven project.

<h3>The steps</h3>

<ol>

<li>Add the remote GitHub repo to your list of Git repos in Eclipse.</li>

<li>In Eclipse, go to File &rarr; Import &rarr; Git &rarr; Projects from Git. (I'm on a Mac; the menu may be a little different for other platforms.)</li>

<li>On the "Select Repository Source", choose "URI".</li>

<li>On the "Source Git Repository" pane, enter the URI info. It might be something like <code>ssh://git@github.com/williewheeler/sip11.git</code>, for example.</li>

<li>On the "Branch Selection" pane, choose the master branch.</li>

<li>On the "Local Destination" pane, decide where you want the local copy to live.</li>

<li>Where it asks you to select an import wizard, choose "Use the New Project wizard" and click "Finish".</li>

<li>Now you have to choose a New Project wizard. Choose Maven &rarr; Maven Project.</li>

<li>From here just create the project like you would any other new Maven project. Once you're done, it will show up in your list of projects in the Package Explorer view, and sharing should be activated.</li>

</ol>
