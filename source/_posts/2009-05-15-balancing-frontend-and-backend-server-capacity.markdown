---
link: http://springinpractice.com/2009/05/15/balancing-frontend-and-backend-server-capacity/
layout: post
title: Balancing frontend and backend server capacity
date: 2009-05-15 14:03:54
comments: true
categories: [Architecture, Chapter 14 - Site-up]
---
In <a href="http://www.pragprog.com/titles/mnee/release-it">Release It!</a>, Michael Nygard explains that a key approach to managing capacity is to balance the capacity of your front-end servers (e.g.,web servers or app servers) with that of your back-end servers. In this brief article we're going to look at what that means and why it's such a good idea to pay attention to this aspect of capacity management.

<h3>A simple example of wasted capacity</h3>

For the sake of example, let's say that we need to design an application that can support up to 100,000 concurrent user sessions within some given response time SLA. The app domain is such that we can partition the user population and employ a horizontally-scalable, sharded design. When there are problems, we want to try to limit the impact to at most 10% of the users. So we decide to go with ten pools that can handle 10,000 concurrent user sessions each. Each pool will have some number of application servers and some number of database servers.

We don't know the number of app and database servers required, but we guess that we're going to need twice as many app servers as database servers. So we try stress testing various pools with that server ratio. Our stress testing reveals that a 6:3 pool can't handle the necessary load but an 8:4 pool can. It's important to us to have fault tolerance, so we decide to add one app node and one database node to the pool. That gives us a 9:5 pool ratio, for a total of 90 app servers and 50 database servers, or 140 servers in all as seen in figure 1:

<img src="http://wheelersoftware.s3.amazonaws.com/articles/balancing-server-capacity/pools.jpg" alt="Figure 1. Server pools" />

So what do you think?

<h3>Improving the utilization of server capacity</h3>

It may seem like we did a good job with this. After all, we defined our load requirements, defined a goal around limiting the impact of outages, created a horizontally scalable design to support the requirements, and then performed stress testing to ensure that our individual pools can handle the required load. So our system ought to be able to meet the stated requirements. But could we have done better?

You may have noticed that in our stress testing of individual pools, we began with an assumption that we'd need twice as many app servers as database servers, but we never actually tested that assumption. Let's say that we repeat our stress testing and discover that even though a 8:4 ratio handles the load, stress-induced failure lies squarely with the database side of the pool. When the pool "fails" (e.g., stops meeting whatever response time SLA we defined),database server CPU is at 90-95% and app server CPU is humming along at 45%. Through further stress testing we discover that we can reduce the ratio to 5:4 while still meeting the overall requirement to service 10,000 concurrent user sessions within the SLA. We still want to be fault-tolerant, so we revise our pool ratio to 6:5, giving us 60 app servers and 50 database servers, or 110 servers total. That's a savings of 30 servers over our previous configuration,or 21.4%, and we get it without having to give anything up. Not bad, except for our hardware vendor!

Here's what's going on. In the first configuration, each pool had three app servers too many. This represents excess capacity in the system. But&mdash;and this is an important point that's easy to miss&mdash; it isn't excess capacity that would allow us to service additional user load. It's inherently unusable capacity. This is because the database tier breaks well before we use that extra app server capacity. So that extra capacity is pure waste. It incurs extra cost (initial purchase price along with operational costs like support, maintenance, rackspace, power,cooling, etc.) and it increases the overall complexity of the system simply by representing additional moving parts that can break or cause (sometimes hard-to-diagnose) problems.

The point bears repeating that the excess capacity is not usable, so it isn't correct to argue that it helps us if our user base grows. We can't tap into it, so it doesn't help. Anyway, the point of the horizontally scalable design here is to support adding capacity by adding pools rather than by growing individual pools.

<h3>Summary</h3>

The bottom line is that proper capacity management involves balancing the capacity of front-end servers with that of the back-end servers. This doesn't mean that the number of app servers needs to be the same as the number of database servers; rather it means that we want the set of app servers and the set of database servers to converge toward failure roughly together. Any systematic delta between the two represents unusable excess capacity and hence undue waste and moving parts.

<div class="endnote">Post migrated from my Wheeler Software site.</div>

