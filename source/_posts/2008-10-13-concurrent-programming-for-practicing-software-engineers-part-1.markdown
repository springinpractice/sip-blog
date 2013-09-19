---
link: http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/
layout: post
title: Concurrent programming for practicing software engineers, part 1
date: 2008-10-13 15:25:07
comments: true
categories: [Tutorials]
---
<div class="intro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/10/15/concurrent-programming-for-practicing-software-engineers-part-3/">Part 3</a></span>
</div>

<a href="http://www.flickr.com/photos/12495774@N02/2405297371/"><img src="http://wheelersoftware.s3.amazonaws.com/articles/concurrent-programming/race.jpg" align="right" /></a>

Many professional software engineers see concurrent programming as something of an arcane art, and indeed it's possible to make it several years into a career without a good grasp of threads. The reasons seem clear enough:

<ul class="square">

<li>Most problems don't require a strong understanding of concurrency. I'd even argue that many web applications, which are often <em>highly</em> concurrent, don't require a strong understanding of concurrency: sometimes there isn't significant data sharing; sometimes the burden of managing concurrency, isolation and deadlocks is pushed back to a transactional data store; sometimes the consequences of races are minor or completely insignificant; etc.</li>

<li>Even when a problem does require some knowledge of concurrency, a lack of knowledge manifests itself only sporadically and generally in ways that are difficult to reproduce, which allows people to mistake their 99.9999% solution for a 100% solution. Even in cases where the developer is under no such delusion, he may believe that his 99.9999% solution "works". He fails to notice that 99.9999% may not be so high when you're talking about long-running applications running on processors that execute billions of cycles per second, and that sometimes the consequences of that 1-in-1,000,000 screwup are <a href="http://en.wikipedia.org/wiki/Therac-25">disastrous</a>.</li>

<li>The "behavioral" structure of a program, as embodied by its threads, is largely orthogonal to the program's "static" structure, as embodied by packages, classes and methods. Since code is organized around the latter, developers can "see" and understand it. Since code is not organized around the behavioral structure (for example, the code does not contain explicit representations of the paths of individual threads), developers have a harder time seeing and understanding it.</li>

</ul>

There are times when you really do need to understand how threads work in order to implement required functionality or else <a href="#">avoid simple but potentially serious mistakes</a>.

<h4>What we'll cover</h4>

In this three-part series of posts we're going to cover several important concurrent programming concepts that every professional software developer needs to understand: serialism, parallelism, race conditions, synchronization, deadlocks and global lock orderings. The concepts form a logical progression that looks a little like an arms race: some problems lend themselves to a parallel solution, but parallelism gives rise to race conditions. We counter race conditions with synchronization (among other techniques), but synchronization leads to deadlocks (among other problems). And so we counter deadlocks with global lock orderings (among other techniques).

You'll note that I said "among other problems" and "among other techniques". The treatment here doesn't pretend to be comprehensive. There are lots of important topics in concurrent programming, such as proper encapsulation, contention-induced (rather than compute-induced) performance issues, read/write locking, minimizing lock scope, fine- vs. coarse-grained locks, timeouts, forking and joining and on and on (and on). I believe however that the topics we're about to treat provide a nice context against which developers can more easily learn other topics. I'll probably write about some of the other topics eventually.

By the end of the series you'll have a basic foundation in multithreaded programming, one that I consider to be required for practicing software engineers.

Our presentation is Java-centric, but developers from other languages (especially C#) should still find the discussion useful.

<h3>What is a thread?</h3>

Let's begin by introducing the star of the show, the thread. The following is probably more accurately considered a description rather than a definition, even though it reads a little like the latter.

A thread is a sequential flow of control with its own program counter and call stack. It shares state, memory and resources with other threads in the same process. Since each thread gets its own call stack, local variables aren't shared. Instance and class variables, however, are shared across threads. It's also possible to define variables that exist outside of methods but are still local to a specific thread, and these are called <a href="http://en.wikipedia.org/wiki/Thread-local_storage">thread-local</a> variables.

OK, so that's what a thread is. What can we do with them?

<h3>Serialism and parallelism</h3>

In software development we're asked to solve different kinds of problem. We may be asked to write code that takes a bunch of information about a loan applicant and produces a verdict as to whether that applicant should get the loan. Maybe we have to write a web-based shopping cart. Or maybe we have to write code to make a guy's arms and legs flail about in a realistic fashion when he falls off a cliff in a video game.

For any given problem there are typically multiple ways to solve it, and different ways to categorize the solutions. One useful way of distinguishing solutions is to separate <em>serial</em> from <em>parallel</em> solutions. Let's look at that distinction in more detail.

<h4>Serial approaches to solving problems</h4>

Some problems have solutions that are essentially a single series of steps. If you're writing a home affordability calculator, for example, the steps might be as follows:

<ol>
<li>Collect data from the user, such as their annual salary, the amount of the down payment, their monthly bill payments, whatever.</li>
<li>Crunch some numbers.</li>
<li>Show the maximum dollar amount the user can afford and an ad for a local Realtor.</li>
</ol>

Our recipe for solving the "home affordability problem" is said to be <em>serial</em> because it's essentially just a series of steps to carry out. We might even go so far as to describing the problem itself as a serial problem, which is just a shorthand way of saying that the most plausible and reasonable solutions to the problem are serial solutions. Either way, the essence of serialism is that we have a series of steps that a single control flow can carry out.

Now let's look at the alternative to serialism, which would be parallelism.

<h4>Parallel approaches to solving problems</h4>

Some problems are amenable to solutions involving multiple concurrent control flows, which is to say that they have <em>parallel</em> solutions. To take a non-computing example, say you have a room with toys scattered all over the floor. You want the room clean. As potential participants to the cleanup effort you have yourself and three kids. A possibly-relevant piece of background information is that one child is entirely responsible for the mess.

Readers with young children will no doubt recognize this classic problem from the field of parenting algorithms. Fortunately, well-known serial and parallel solutions are available. The best approach in any given case depends strongly upon your goals:

<ol>

<li>If you're trying to emphasize fairness and personal responsibility, you might opt for the obvious serial solution even though it's slower. (In reality this isn't a serial solution; it inevitably involves close supervisory involvement by a supervisory parent thread. We can ignore that detail.)</li>

<li>If you're in a big hurry because you need to get the kids off to school, you might choose the obvious parallel solution, despite the unfairness.</li>

<li>If you're in an even bigger hurry you might choose the "other" serial solution. (Hint: it doesn't involve any kids.)</li>

</ol>

Whether we're talking about serialism or parallelism, there is a recurring concept, which is that of a flow of control. In the serial case we have exactly one flow. In the parallel case we have more.

Computer scientists, hardware engineers and software engineers spend a lot of time with the distinction between serialism and parallelism. Let's look at why that is.

<h4>Serialism vs. parallelism: why do we care?</h4>

In a word, performance. In many cases&mdash;not all, but many&mdash;parallel solutions are simply faster than serial solutions, owing to the "many hands make light work" effect. And from a practical perspective, that's the main reason we care about concurrency (i.e., parallelism) despite its many attendant complications.

The <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">next post in our series</a> explains what some of those complications are and some ways to overcome them.

<div class="outro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/10/15/concurrent-programming-for-practicing-software-engineers-part-3/">Part 3</a></span>
</div>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
