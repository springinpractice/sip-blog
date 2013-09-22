---
layout: post
title: "Closed loops: the secret to collecting configuration management data"
date: 2012-01-03 12:45:19
comments: true
categories: [Architecture, Chapter 11 - CMDB]
---
Hi all, Willie here. Happy New Year!

In my last post, [Devops: How NOT to collect configuration management data](http://springinpractice.com/2011/12/26/devops-how-not-to-collect-configuration-management-data/), I gave a quick rundown of some losing CM data approaches that I and others have attempted in the past. Most of these approaches were variants of asking people for information, putting their answers in documents somewhere and never looking at the documents again.

This time around I'm going to describe a key breakthough that our team finally made--one that made it a lot easier to collect and update the data in question.

That breakthrough was the concept of a *closed loop* and how it relates to configuration management.

[As it happens, the concept is well-known in configuration management circles, but at the time it wasn't well-known to us. So we discovered something that other people already knew. It's in that limited sense I say we made a breakthrough.]

We're going to have to build up to the closed loop concept. Let's start by looking at who has the best CM data in the org.

Who has the best CM data?
-------------------------

Different orgs are different, so it's tough to make blanket statements about who has the best CM data. But what I can do is give the answer for my workplace, and hopefully the principles will make sense even if the reality is different where you work. But I'll bet for a lot of orgs it's quite similar.

To avoid keeping you in suspense, the answer is...

**Winner: the release team.** Where I work, the release team has the best CM data, where "best" means something like comprehensive, accurate and actively managed. The release team knows which apps go on which servers, which service accounts to launch Tomcat or JBoss under, which alerts to suppress during a deployment, and so on. They know these things across the entire range of apps they support. It's all documented (in YAML files, databases, scripts, etc.) and it's all maintained in real time.

Let's look at some other teams though.

* **App teams have nonsystematic data.** The app teams typically know the URLs for their apps, which servers their apps are on (or at least the VIPs), the interdependencies between their apps and other adjacent systems (web services, databases). But the knowledge is less systematic. It's more like browser bookmarks, tribal knowledge and not-quite-up-to-date wiki pages. And any given developer knows his own app and maybe the last app or two he worked on, but not all apps.****
* **Ops teams have to depend on busy developers for info.** The ops teams have better or worse information depending on how close to the app teams they are. The team in the NOC is almost entirely at the mercy of the app teams to write and update knowledge base articles. As you might imagine, it can be a challenge to ensure that developers up against a deadline are writing solid KB articles and maintaining older ones. For the NOC it's very important to know who the app SMEs are, given such challenges. Even this is not always readily clear as org changes occur, new apps appear, old apps get new names and so on.
* **App support teams are more expertise-driven than data-driven.** The app support teams (they handle escalations out of the NOC) are generally more familiar with the apps themselves and so build up stronger knowledge about the apps, but this knowledge tends to be stronger with "problem child" apps. Also, different people tend to develop expertise with specific apps.

Why does the release team have the best CM data?
------------------------------------------------

The release team has the best CM data because properly maintained CM data is fundamental to their job in a way that it's not for other teams.

> First, a quick aside. If your company isn't regulated by [SOX](http://en.wikipedia.org/wiki/Sarbanes%E2%80%93Oxley_Act), you may be wondering about what a release team is and why we have one. Among many other things, SOX requires a separation of duties between people who write software and people who deploy/support it in the production environment. The release team's primary responsibility is to release software into production. We actually have a couple of release teams, and each of them services many apps. It would not be feasible from a cost and utilization perspective for each app team to have its own dedicated release engineer. The release teams release software at low-volume times, generally during the wee hours.

Back to this idea that the release team needs proper CM data more than the other teams do. Why am I saying that?

![Scrum](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-03-closed-loops-the-secret-to-collecting-configuration-management-data/scrum.jpg)

**Here's why.** The software development team is highly motivated to release software at a regular cadence. A fairly limited number of release engineers must service hundreds of applications (generally not all at once, though), so "tribal knowledge" isn't a viable strategy when it comes to knowing what to deploy where. It must be thoroughly and accurately documented. Releases happen every week, and late at night, so it's not reasonable for the release team to call up his buddy on the app team and ask for the list of app servers. The release team needs this information at their fingertips. If they don't have it, the software organization fails to realize the value of its development investment.

Indeed, "documented" is the wrong word here, because deployment automation drives the deployments. The CM data must be properly "operationalized", meaning that it must be consumable by automation. No Word docs, no Excel spreadsheets, no wiki pages. More like YAML files, XML files, web service calls against a CMDB, etc.

Importantly, when the data is wrong, the deployment fails. People really care about deployments not failing, so if there are data problems, people will definitely discover and fix them.

Let's look at the app and ops teams again.

* **App teams can make do without great CM data.** The dependency of app developers on their CM data is softer. Yes, a developer needs to know which web services his app calls, but someone just explains that when he joins the project, and that's really all there is to it. If he has a question about a transitive dependency, he might ask a teammate. If he needs to get to the app in the test environment, he probably has the URL bookmarked and the credentials recorded somewhere, but if not, he can easily ask somebody. 99% of the time, the developer can do what he needs to do without reference to specific CM data points. The developer may or may not automate against the CM data.
* **Ops/support teams need good CM data, but expertise is cheaper in the short- to medium-term.** Except in cases involving very aggressive SLAs, even ops often has a softer dependency on CM data than the release team does. Since (hopefully) app outages occur much less frequently than app deployments, the return on knowledge base investments is more sporadic than that on deployment automation. If the app in question isn't particularly important, investments in KB articles may be very limited indeed. In most cases, investing in serious support firepower (when something breaks, bring significant subject matter expertise to bear on the problem) yields a better short- to medium-term return. (Of course, in the longer term this strategy fails, because eventually there will be the very costly outage that takes the business out for several days. That's a subject for a different day.)

Now we're in a good place to understand closed loops and why they're so important for configuration management data.

Closed loops and why they matter
--------------------------------

I think of closed loops like this. There's a "steady state" that we want to establish and maintain with respect to our CM data. We want it to be comprehensive, accurate and relevant. When the state of our CM data diverges from that desired steady state, we want feedback loops to alert us to the situation so we can address it. That's a closed loop.

**Example 1: deployment automation.** The best example is the one that we already described: deployment data. Deployment data drives the deployment process, and when the data is wrong, the deployment process fails. Because the deployment process is extremely important to the organization, some level of urgency attaches to fixing wrong data. But it's not just wrong data. If we need to deploy an app and there's missing data in the CMDB, then sorry, there's no deployment! Rest assured that if the deployment matters, the missing data is only a temporary issue.

**Example 2: fine-grained access controls.** Here's another example: team membership data. We've already noted that for operational reasons it's very important to know who is on which development team. This isn't something that's going to be in the HR system, and people have better things to do than to update team membership data. But what happens when that team membership data drives ACLs for something you care about being able to do, like deploying your app to your dev environment? Now you're going to see better team membership data.

The basic concept is to find something that people really, really care about, and then make it strongly dependent on having good CM data:

![Closed loop](http://springinpractice.s3.amazonaws.com/blog/images/2012-01-03-closed-loops-the-secret-to-collecting-configuration-management-data/closed_loop.png)

Ideally it's best if the CM data drives an automated process that people care about, but that's not strictly necessary. In my org, for instance, there's a fairly robust but manual goal planning and goal tracking process. Every quarter the whole department goes through a goal planning process (my goals roll up into my boss' goals and so on), and then we track progress against those goals every couple of weeks. The goal planning and tracking app requires correct information about who's on which team, and so this serves to establish yet another closed loop on the team membership data. It also illustrates the point that you can hit the same type of data with multiple loops.

Design your CM strategy holistically
------------------------------------

There are several areas in technology where it pays to take a holistic view of design: security, user experience and system testing come immediately to mind. In each case you consider a given technical system in its wider organizational context. (Super-duper-strong password requirements don't help if people have to write them down on Post-Its.)

Configuration management is another place where it makes lots of sense to take a holistic approach to design. For any given type of data (there's no one-size-fits-all answer here), try to figure out something important that depends on it, and then figure out how to tie that something to your data so that wheels just start falling off if the data is wrong, incomplete and so on. Again, data-driven automated processes are superior here, but any important process (automated or not) will help.

Fewer meetings?
---------------

Almost forgot. In the last post, I mentioned that I'll equip you to get out of some pointless meetings. The meetings in question are the ones where somebody wants to get together with you to collect CM data from you so they can post it to their Sharepoint site. Decline those--they're just a waste of time. Insist that people be able to articulate the closed loops that they will be creating to make sure that someone discovers gaps and errors in the data. I've been in plenty of such meetings, and in some cases they're set up as half- or full-day meetings. I don't do those anymore.
