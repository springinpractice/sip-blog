---
link: http://springinpractice.com/2008/10/01/some-thoughts-on-the-new-springsource-maintenance-policy/
layout: post
title: Some thoughts on the new SpringSource maintenance policy
date: 2008-10-01 22:38:35
comments: true
categories: [Random Thoughts]
---
At the risk of getting myself involved in a highly charged political issue, I have some thoughts about the <a title="new SpringSource maintenance policy" href="http://www.springsource.com/products/enterprise/maintenancepolicy">new SpringSource maintenance policy</a> that I wanted to share. Not sure that I know what I'm talking about, but I have no doubt that others will let me know if I'm totally off-base.

Initially I was a little disappointed when I heard the news about the change. I don't have anything against capitalism, and I think that the Spring developers absolutely ought to benefit financially from all the good work that they've been doing over the years. The disappointment was really just a function of my being a bit of a cheapskate: I'm happy to contribute on the forums, contribute JIRA issues, even the occasional code patch, but I suppose I've grown too used to being able to download the latest and greatest for free.

But I found out that I misinterpreted the part about "major" releases in an important way, and now I feel a lot better about things.

I thought that SpringSource was saying that the community would be able to download binaries for 3.0, 4.0, 5.0 etc. for free, as well as 3.x (x &gt; 0) releases as long as the 3.x releases were released within the first three months of the 3.0 release.

I'm happy to report that that's totally wrong.

It turns out that the community will have access to binaries for 3.0, 3.1, 3.2, 3.3, etc., as they appear. The for-pay releases will be the 3.1.x (x &gt; 0) releases that come out three months after the initial 3.1 release. Don't take my word for it though. It's <a title="Rod Johnson interview" href="http://java.dzone.com/news/qa-with-rod-johnson-over-sprin">right here</a> and <a title="FAQs" href="http://www.springsource.com/products/enterprise/maintenancepolicy/faq">here too</a>.

I can definitely live with that. Sure, I'd prefer to download 3.1.5 for free, but it's not going to kill me to wait for 3.2. I think that there are probably a lot of developers who would feel the same way.

I have to wonder, though, whether SpringSource might do a better job explaining the change. It's pretty easy to read
<blockquote>After a new major version of Spring is released, community maintenance updates will be issued for three months to address initial stability issues.  Subsequent maintenance releases will be available to SpringSource Enterprise customers.  Bug fixes will be folded into the open source development trunk and will be made available in the next major community release of the software.</blockquote>
and think that you're going to have to wait a year for an update. I interpret "major version" as meaning Spring 2, Spring 3, etc. Maybe that's not the way SpringSource thinks of major versions (though that's news to me), but it doesn't really matter what SpringSource thinks - what matters is what the Spring community thinks, and I can't be the only one who read the above passage thinking that we'd have to pay for version 3.3.

My understanding is that this change is about getting enterprises to start paying for Spring while letting smaller shops and individual developers still have access to pretty recent stuff. Enterprises are slower about upgrading versions. So say you're an enterprise and you built an app against version Spring 3.0.2. It's more painful for you to upgrade to version 3.1 than it is to just buy the enterprise license and get your 3.0.x upgrades, so you buy the license. On the other hand, smaller organizations are in many cases fine with going from 3.0.2 to 3.1, as are many individual developers (indeed, I think a lot of individual developers eagerly await new releases), so they can still use it for free without too much inconvenience. No doubt some smaller organizations and developers will be inconvenienced though.

If my understanding is correct, I think SpringSource ought to consider explaining it in roughly that way instead of this:
<blockquote>"The maintenance policy provides IT teams responsible for building and running mission-critical production systems with the quality, assurance and peace of mind that comes from reliable, predictable and guaranteed support direct from the source,” said Mark Brewer, vice president of enterprise delivery for SpringSource. “And, the policy will always give the developer community access to the same cutting-edge innovations they have come to expect with Spring."</blockquote>
When developers read the above, they hear "blah blah blah." Developers hate that kind of explanation. I assume that since Brewer is VP of enterprise delivery he's addressing enterprise customers more than he is the community, but (1) it's important to address the community since you don't want people defecting from the platform, and (2) there's a way to present the explanation so that it's palatable to enterprises. I am myself in corporate IT leadership, and if somebody told me that the enterprise license allows you to get bugfixes without having to upgrade from 3.0.x to 3.1, I would be able to see value in that.

Anyway, I'm glad to discover that the situation is not what I had originally thought, and to my mind SpringSource is striking a reasonable balance between their desire (and right) to build a profitable business around Spring vs. their desire to maintain a supportive developer community. They just need to work on the communication. :-)