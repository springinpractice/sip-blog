---
layout: post
title: "About our book"
date: 2008-08-31 05:20:06
comments: true
categories: [News]
---
<a href="http://www.flickr.com/photos/seeks2dream/2213191214/"><img class="size-full wp-image-49 alignnone" src="http://springinpractice.com/wp-content/uploads/2008/08/2213191214_8db8aba33c2.jpg" alt="" width="500" height="222" /></a>

<strong>In a nutshell:</strong> <em>Spring in Practice</em> is an applied book on the Spring Framework. It's not theoretical, and it's not a reference manual.

Let's look at that statement in more detail, since we'd like you to know what you're buying before you buy it. And maybe we can even pique your interest in what we're doing.
<h3>We assume you already have your reasons for using Spring</h3>
While it's true that we like Spring, we assume that you already have your reasons for using it - maybe you heard it was a good framework and you want to try a few things out, or maybe you've just joined a development team where it's already been decided by the powers that be that you're going to be using Spring.

Given that assumption, we don't make much of an effort to "sell" the framework.  We don't rehash the history, and we don't explain in much detail what recommends Spring over this this-or-that alternative.  Instead we give you some bare minimum technical background (mostly around practices that we adopt in the book, such as using the "p" namespace or using annotations to accomplish various things) and then launch immediately into the "how-to" materials.

We try to show you not only areas in which Spring excels, but also areas in which Spring doesn't help you out very much or even areas in which Spring seems to make things more difficult than they ought to be.  With respect to the latter, we believe that such information is extremely practical but not necessarily easy to find.
<h3>It's not a reference manual</h3>
In saying that <em>Spring in Practice</em> is an applied book, we mean to distinguish it from reference manuals, which generally take a technology-centric perspective in that they organize around the technologies that make up their topic.  For example, reference manuals on the Spring Framework typically start with a history of the framework, and then go on to explain why Spring represents an improvement over traditional approaches. They then continue with chapters on the IoC container, data access, transactions, web services, Spring Web MVC, Spring Web Flow, Spring Security, testing and so forth.

Those are certainly appropriate reference manual topics, but the problem is that it's hard for a novice to become productive without first absorbing a lot of background information.  To build a single vertical in a web application, such as a user registration form (including all aspects of the form - web presentation, form validation, form processing and storage, security, web flow if necessary, etc.), you potentially have to read several chapters' worth of material.  It isn't necessarily obvious what you can skip (for example, can you skip the chapter on Spring Web Flow? can you skip the section on programmatic validation?), and it isn't necessarily clear how all the pieces fit together.  The novice must analyze each chapter to understand what is and isn't relevant, and then must further synthesize the relevant pieces into a coherent solution.

In a certain respect, the novice's difficulties are exacerbated by the fact that reference manuals of necessity tend to present toy examples.  On the one hand, the simplification is useful because it allows the reader to grasp the technology without getting bogged down in messy problem domain details.  On the other hand, most readers - novices included - are tasked with solving realistic problems, and so there's a gap between the reader's task and what the book directly explains.

Reference manuals are optimized for people who already have a reasonable understanding of the technology and who just need to learn or remember how a certain piece works.  As such they serve an important function; certainly all readers, novices again included, will benefit from having a reference manual handy.  But novices relying solely on a reference manual will usually read a lot of material that isn't directly relevant to what they're trying to do, and spend lots of time figuring how to integrate the pieces.
<h3>We organize around problems rather than around Spring technologies</h3>
<em>Spring in Practice</em> takes a completely different approach.  We wanted to contribute something optimized for people who are newer to Spring and who, despite their novice status, are required to solve realistic problems, and quickly.

Except for the first few introductory chapters, the book is organized not around Spring Framework technologies, but instead around <em>problems</em> that you might want or need to solve using Spring.  For instance, you won't find anything in the table of contents that says "Integrating Hibernate and Spring" or "Using Spring Web MVC to do such-and-such;" rather we treat problems that can be stated in the language of application domains.  Some of the problems are highly general in that they show up in multiple application categories; examples include the following:
<ul>
	<li>Display a user registration form</li>
	<li>Store user passwords securely</li>
	<li>Allow users to select a site theme</li>
	<li>Use CAPTCHAs to prevent comment spam</li>
	<li>Create a business process monitor (BPM)</li>
</ul>
Other problems are more category-specific (but not tied to any particular industry). Again, here are some examples:
<ul>
	<li><strong>Marketing:</strong> Create HTML e-mail templates for e-mail marketing campaigns</li>
	<li><strong>Lead generation:</strong> Submit leads to a web service</li>
	<li><strong>E-commerce:</strong> Implement a simple shopping cart</li>
	<li><strong>CRM:</strong> Implement contact history</li>
</ul>
Because we focus on domain problems instead of technologies, we can do a few things that are harder for reference manuals to do:
<ul>
	<li><strong>We can treat problems more realistically.</strong> Instead of including only those pieces of the problem that scaffold the technical discussion, we can include other concerns that are often neglected in reference-type treatments, including issues around usability, security, search engine friendliness, and so on.  Often the concerns in these non-functional areas give rise to requirements that further drive the technical discussion.  Our hope is that readers - novices and veterans alike - will find this aspect of our book to be of great value.</li>
	<li><strong>We can focus the solution on those pieces that are directly relevant to the problem.</strong> If user registration forms deal with hashes and salts but not ACLs, then you'll read about hashes and salts and you won't read about ACLs.</li>
	<li><strong>We can integrate all aspects of the solution in a single place.</strong> You won't have to "put it all together;" we've already done that for you.</li>
</ul>
Hopefully that gives you a good understanding of the book we're writing.  If you have thoughts or suggestions please do let us know.