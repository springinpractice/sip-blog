---
layout: post
title: "Devops: How NOT to collect configuration management data"
date: 2011-12-26 22:31:55
comments: true
categories: [Architecture, Chapter 11 - CMDB]
---
Hi all, Willie here. This time we're going to step away from the keyboard and get architectural. But no ivory towers here. In my next two blog posts, I'm going to give you something that will get you out of lots of pointless meetings.

Got your attention yet? Good!

If you're in devops, one of the things that you have to figure out is how to collect up all the information that will allow you to manage your configuration and also keep your apps up and running. This isn't too hard when you have two apps sitting on a single server. It's much harder when you have 400 apps deployed to thousands of servers across multiple environments and data centers.

So how do you do that? Let's start off by looking at some things that just don't work. In my next post I'll share with you an approach that <em>does</em> work.
<h3>Some quick background</h3>
Probably owing to some psychological glitch, I've always been an information curator. Especially when I was in management, it was always important to me to understand exactly where my apps were deployed, which servers they were on, which services and databases they depended on, which servers <em>those</em> services and databases lived on, who are the SMEs for a given app, and so on.

If you work in a small environment, you're probably thinking, "what's the big deal?"

Well, I don't work in a small environment, but the first time I undertook this task, I probably would have said something like, "no sweat." (That's the developer in me.) Anyway, for whatever reason, I decided that it was high time that somebody around the department could answer seemingly basic questions about our systems.

<h3>Attempt #1: Wiki Wheeler</h3>

<a href="http://www.flickr.com/photos/72213316@N00/5533149173/sizes/s/in/photostream/"><img class="alignright size-full wp-image-244" title="bullwhip" src="http://springinpractice.com/wp-content/uploads/2011/12/bullwhip1.jpg" alt="Wiki Wheeler" width="280" height="351" /></a></strong>

The first time I made the attempt (several years ago), I was a director over a dozen or so app teams. So I created a wiki template with all the information that I wanted, and I chased after my teams to fill it out. My zeal was such that, unbeknownst to me, I acquired the nickname "Wiki Wheeler". (One of my managers shared this heartwarming bit of information with me a couple years later.) I guess other managers liked the approach, though, because they chased after their teams too.

This approach started out decently well, since the teams involved could manage their own information, and since I was, well, Wiki Wheeler. But it didn't last. Through different system redesigns and department reorgs, wiki spaces came and went, some spaces atrophied from neglect, and there was redundant but contradictory information everywhere. The QA team had its own list, the release guys had their list, the app teams had their list. The UX guys might have even gotten in on the act. Anyway, after a year or so it was a big mess and to this day our wiki remains a big mess.

<h3>Attempt #2: My huge Visio</h3>

<strong></strong>The second time, my approach was to collect the information from all the app development teams. We had hundreds of developers, so to make things efficient, I sent out a department-wide e-mail telling everybody that I was putting together a huge Visio document with all of our systems, and I'd appreciate it if they could reply back with the requested information. And while I had to send out more nag e-mails than I would have liked, the end result was in fact a huge Visio diagram that had hundreds of boxes and lines going everywhere. I was very proud of this thing, and I printed out five or six copies on the department plotter and hung them on the walls.

How long do you think it was before it was out of date?

I have no idea. I seriously doubt that it was ever correct in the first place. Nobody (myself included) ever used the diagram to do actual work, and its chief utility was in making our workplace look somewhat more hardcore since there were huge color engineering plots on the walls.

There was one additional effect of this diagram, though I hesitate to call it utility. I acquired the wholly undeserved reputation for knowing what all these apps were and how they related to one another. This went on for years through no fault of my own. I mention it because it's relevant for the next attempt.

<h3>Attempt #3: Disaster recovery</h3>

<a href="http://www.flickr.com/photos/55123857@N00/250743228/"><img class="alignright size-full wp-image-253" title="asteroid" src="http://springinpractice.com/wp-content/uploads/2011/12/asteroid.jpg" alt="When asteroids strike..." width="240" height="164" /></a>This one wasn't my attempt, but it's still worth describing since I have to imagine that we aren't the only ones who ever tried it.

As a rapidly growing company, disaster recovery became an increasingly big deal for us several years back, and there was a mandate from up high to put a plan in place. This involved collecting up all the information that would allow us to keep the business running if our primary data center ever got cratered. The person in charge of the effort spent about two years meeting with app teams or else "experts" (me, along with others who, like me, had only the highest-level understanding of how things were connected up) and documenting it on a Sharepoint site where nobody could see it.

This didn't work at all. Most people were too busy to meet with her, so the quality of the information was poor. Apps were misnamed, app suites were mistaken for apps, and to make a long story short, the result was not correct or maintainable.

<h3>Attempt #4: Our external consultants' even bigger Visio</h3>

Following a major reorg, we brought in some external consultants and they came up with their own Visio. By this time I had already figured out that interviewing teams and putting together huge Visios is a losing approach, so I was surprised that a bunch of highly-paid consultants would use it. Well, they did, and their diagram was, well, "impressive". It was also (to put it gently) neither as helpful nor as maintainable as we would have liked.

<h3>Attempt #5: HP uCMDB autodiscovery</h3>
We have the HP uCMDB tool, and so our IT Services group ran some automated discovery agents to populate it with data. It loaded the uCMDB up with tons of fine-grained detail that nobody cared about, and we couldn't see the stuff that we actually did care about (like apps, web services, databases, etc.).

<h3>Attempt #6: Service catalog</h3>
Our IT Services group was pretty big on ITIL, so they went about putting together a service catalog. The person doing it collected service and app information into an Excel spreadsheet and posted it to a Sharepoint site. But this didn't really get into the details of configuration management. It was mostly around figuring out which business services we offered and which systems support which services.

They ended up printing out a glossy, full-color service catalog for the business, but nobody was really asking for it, so it was more of a curiosity than anything else.

There were other attempts too (Paul led a couple that I haven't even mentioned), but by now you get the picture.

<h3>So why did these attempts fail?</h3>
To understand why these attempts failed, it helps to look at what they had in common:
<ul>
	<li>First, they generally involved trying to collect up information from SMEs and then putting it in some kind of document. This could be wiki pages, Visio diagrams, Word docs on Sharepoint or an Excel spreadsheet.</li>
	<li>Once the information went there, nobody used it. So the information, if it was ever correct and/or complete in the first place, was before long outdated, and there wasn't any mechanism in place to bring it up to date.</li>
</ul>
In general, people saw the problem as a data <em>collection</em> problem rather than a data <em>management</em> problem. There needs to be a systematic, ongoing mechanism to discover and fix errors. None of the approaches took the management problem seriously at all--they all simply assumed that the organization would care about the data enough to keep it up to date.

In the next installment, I'll tell you about the approach we eventually figured out. And as promised, that's also where I'll explain how to get rid of some of your meetings.

<span class="icon stickyNote"><em>Interested in configuration management? I'm working on an open source CMDB called <a href="http://zkybase.org/">Zkybase</a>.</em></span>
