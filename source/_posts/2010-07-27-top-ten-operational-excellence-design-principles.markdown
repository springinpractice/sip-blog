---
link: http://springinpractice.com/2010/07/27/top-ten-operational-excellence-design-principles/
layout: post
title: Top ten [and counting] operational excellence design principles
date: 2010-07-27 01:05:26
comments: true
categories: [Architecture, Chapter 14 - Site-up]
---
<a href="http://www.flickr.com/photos/kim_scarborough/147973192/"><img src="http://springinpractice.s3.amazonaws.com/blog/images/noc.jpg" alt="Operational excellence" align="right" /></a>

Here are some key operational excellence design principles. These are intended for system architects (both on the software and infrastructure side) and operational staff.

This started out as a top ten list, but it's been growing.

<strong>1. Organizational focus.</strong> OK, this one isn't a design principle, but it's too important to neglect. Having an organizational focus on operational excellence really matters. Everything from isolating core development from production support (that is, create a group within development whose daily responsibility is to insulate core development from production), creating a top-ten hit list and just working that weekly, setting explicit availability and performance goals and then tracking to them&mdash;speaking from personal experience, those are things that can create dramatic and almost immediate results.

<strong>2. Aggressively automate processes and controls.</strong> Automate ticket workflow, local builds, integration builds, tests, provisioning, OS installs, patching, deployments, rollbacks, monitoring, alerting, diagnostics, escalation, backups, log rotation, reporting, auditing&mdash;everything you can! Automation increases speed and limits variance, which is an operations-killer.

On the control side, in cases where it's possible to use a technical means to enforce policy (automation, tightly constrained configuration schemas, access controls, etc.), use it. The ideal control makes the undesirable action or state technically impossible.

<strong>3. Design for supportability.</strong> The NOC should have the tools and information necessary to resolve at least 80% of production issues without directly involving the development team. Put the right monitoring, diagnostic and management (e.g., SNMP, JMX) capability in place.

<strong>4. High availability is all about redundancy.</strong> In general high availability involves redundancy at all levels: power sources, network interfaces, switches, RAIDs, RAC nodes, Internet pipes and even data centers if you want to withstand site failures. Redundancy means that you've deployed independent, readily-available (perhaps active) capacity beyond what's normally needed to hit performance SLAs. Without this extra capacity, component failures often create complete system unavailability through a chain reaction mechanism that looks like this: (1) one component fails; (2) other components take on the excess demand, creating performance issues; (3) other components fail under load; and (4) system is unavailable.

Don’t confuse high availability with load balancing. If all <em>n</em> instances in a load-balanced pool are required to meet a given performance SLA, then there is no redundancy, because there's no spare capacity to take up the slack in the event of a node failure. Indeed exposure is substantially <em>increased</em>, because all it takes is any one node to fail, and your pool is at best missing performance SLAs, and at worst rendered completely unavailable as described above.

<strong>5. Control failure modes.</strong> Components will fail. Failure modes should be designed, not emergent or accidental. Design failures to be transparent and limited in impact, using techniques such as <a href="http://springinpractice.com/2010/07/06/annotation-based-circuit-breakers-with-spring/">circuit breakers</a>. Test the design to ensure that it behaves as expected. Create automation and knowledge base articles to target specific failure modes for fast remediation.

<strong>6. Prefer horizontal approaches to scalability.</strong> You will sometimes need to add capacity, either to accommodate business growth or else to handle production issues. Prefer horizontal scalability to vertical scalability, since vertical scalability is more tightly coupled to technology. The database is often the constraint here, so consider approaches that support horizontal scalability, such as data sharding, separating reads from writes and farming out reads, and non-relational data stores.

<strong>7. Virtualize aggressively.</strong> Virtualization makes it easier to enforce standards, control drift, add capacity, increase utilization and in general create configuration options.

<strong>8. Capacity management = capacity planning + JIT capacity.</strong> Capacity planning is an art, not a science, and it's not always possible to anticipate exactly how much capacity you will need. There's a balance to strike between ensuring sufficient capacity and using resources efficiently, and sometimes the drive to be efficient will result in underestimates. In such situations, a JIT capability is key.

For rollouts (e.g., new apps, new features, betas, promotions, or anything that might reasonably impact demand in a material way), it's better to overallocate and then scale back than it is to underallocate and scale up. Both approaches use resources efficiently, but only the former will allow you to meet aggressive SLAs. Virtualization helps tremendously here, partly because it pools excess physical capacity for shared use (thus limiting cost), and partly because it makes reaping excess capacity during scale-backs much easier.

<strong>9. Account for operational excellence overhead costs.</strong> Treat overheads such as monitoring and virtualization as costs that the design needs to account for, not as an excuse to neglect key operational requirements. Deal with instrumentation overhead, for example, by being selective about what you instrument. If your instrumentation adds 1 ms to a 200 ms call, that may not be a big deal. If it adds 1 ms to a 5 ms call inside a tight loop, that may be a bigger problem. 

<strong>10. Focus monitoring efforts on ensuring that the business is running.</strong> It doesn’t matter whether a particular CPU is running at 90% utilization if you can’t connect that to a business or customer impact. Start by identifying the metrics that the business or customer cares about, and monitor those to ensure that the numbers are within expected bounds. Work backwards to correlated technical metrics.

<h3>Bonus principles</h3>

<strong>11. Defense in depth applies to operational excellence too.</strong> A well-known security principle, defense in depth applies to operational excellence too. Automation controls configuration drift, but so does keeping people off the servers, being able to refresh the environment on demand, and using a configuration schema that makes certain types of drift impossible. Layer your defenses.

<strong>12. Treat support processes and staff as part of the system.</strong> All the monitoring, diagnostics and knowledge base articles in the world won’t help if the support staff doesn’t know how to use them. Test the human component of the system (training, communication, etc.) on a regular basis to ensure that when problems arise, the human response will be the one that was designed.

The next three principles are courtesy of Satish Menon. Thanks Satish.

<strong>13. Design for graceful degradation:</strong> One example, if load increases, serve out of stale cache (where it is appropriate) rather than go to the origin server (database or whatever). We added a mechanism to Squid at Yahoo to give hints on what can be served stale.

<strong>14. Design for independent scalability (this conflicts your horizontal vs. vertical scalability).</strong> I am saying you need both. If there is a heavy hit on one of the services (e.g. Gradebook) over other, isolating the pathway to that in the design would allow you to scale that one service without scaling all of the rest of the infrastructure.

<strong>15. Put data in front of engineers.</strong> Define and measure SLAs for each of the major server interactions and put dashboards in place so people can see how their subsystems are doing.

Here's another one that I find myself repeating often. Don't know how I missed it the first time around.

<strong>16. Minimize the number of moving parts.</strong> More moving parts mean that more things can break. Better for example to templatize your middleware into a VM template and instantiate than to instantiate a bunch of bare-bones VMs and then follow up with a a lot of middleware installations. You can even get aggressive and take the same approach to app deployments (i.e., templatize a build that includes the app to be released). All things being equal, designs should try to minimize the number of things that can go wrong.

<h3>References</h3>

<ul>
	<li><a href="http://www.pragprog.com/titles/mnee/release-it">Release It!</a>, by Michael Nygard (Pragmatic)</li>
	<li><a href="http://www.amazon.com/Scalable-Internet-Architectures-Theo-Schlossnagle/dp/067232699X">Scalable Internet Architectures</a>, by Theo Schlossnagle</li>
	<li><a href="http://jprall.vox.com/library/post/85-operations-rules-to-live-by.html">(85) Operations Rules to Live By</a>, by Jon Prall</li>
</ul>

