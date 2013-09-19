---
layout: post
title: "Concurrent programming for practicing software engineers, part 3"
date: 2008-10-15 16:06:02
comments: true
categories: [Tutorials]
---
<div class="intro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/10/15/concurrent-programming-for-practicing-software-engineers-part-3/">Part 3</a></span>
</div>

This third and final post in our series on concurrency explains how synchronization mistakes can lead to the problem of deadlocks, and what you can do to minimize or eliminate them.

<h3>How synchronization gives rise to deadlocks</h3>

We saw in the <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">previous post</a> how a thread will grab a lock just before entering a critical section, thereby forcing other threads to "block" (concurrency lingo for "wait for another thread to release a lock") until they can acquire the lock. Normally this blocking behavior is unremarkable. It is a standard part of concurrent programming since we're trying to protect data from data races.

If there's too much contention for locks, that can of course have a negative effect on performance since threads are sitting around waiting instead of doing work. But usually this just means subpar performance rather than hanging.
    
The pathological case of contention for locks is <em>deadlocking</em>. This is characterized by two or more threads becoming permanently hung. They are notoriously difficult to predict, avoid and handle, but it's not impossible. The secret to understanding deadlocks is to understand that they arise when threads wait for each other to release a lock in a cyclic fashion. The simplest case would be thread T<sub>1</sub> waiting for T<sub>2</sub> to release a lock, and T<sub>2</sub> waiting for T<sub>1</sub> to release a lock. Neither one can make forward progress. The next simplest case is T<sub>1</sub> waiting for T<sub>2</sub>, T<sub>2</sub> waiting for T<sub>3</sub>, and T<sub>3</sub> waiting for T<sub>1</sub>. Now we have three deadlocked threads. Obviously the cycle can be of arbitrary length; all threads involved will be deadlocked.
    
Here's a diagram that shows how it works:

<div style="margin: 20px auto; width: 584px;">
    <div><img src="http://wheelersoftware.s3.amazonaws.com/articles/concurrent-programming/deadlocks.jpg" alt="The cyclic nature of deadlocks" /></div>
    <div class="caption"><span class="figure-label">Figure 1.</span> Deadlocks arise from circular dependencies between threads.</div>
</div>

In figure 1, you can see that each thread holds the lock needed by the preceding thread. As the dependencies are circular, there is no way for any of the threads to make forward progress once something like the above happens. All five threads are hung&mdash;that is, deadlocked.

<div style="margin-right: 10px; float: left">
    <div><img src="http://wheelersoftware.s3.amazonaws.com/articles/concurrent-programming/vambrace-m16.jpg" alt="" /></div>
    <div class="photo-caption" style="text-align: center">Photo credit: <a href="http://www.flickr.com/photos/gladius/2296372699/">gladius</a></div>
</div>

<h4>A furry aside</h4>

By now you might understand what I meant at the beginning of the article when I said that concurrent programming is kind of like an arms race. We start with single threads, but sometimes that's too slow, so we introduce multithreading. That causes a problem though, which is that the threads can interfere with one another when working with shared data. So we introduce turn-taking through mutexes (and through the <code>synchronized</code> keyword in particular). But that has the effect of limiting parallelism, and if we take that too far, we can end up sacrificing the parallelism (and performance) that we sought in the first place. In the most extreme case, we can inadvertently cause threads to deadlock.
    
(Maybe <a href="http://en.wikipedia.org/wiki/Whac-a-mole">Whac-A-Mole</a> is a better metaphor. That's OK. I'm going to stick with arms race just because I really like the mech/cyber photo. Think of it as an arms race between software engineers and small furry rodents.)

<div style="clear: both"></div>

Now let's consider an example of code vulnerable to deadlocks.

<h4>Some deadlocky code</h4>

For this one we're going to use the standard bank account funds transfer example. Yes, it's a clich&egrave;d example, but it's such a good example and originality is overrated anyway. Here's listing 3:

<div>
    <div><span class="code-listing">Listing 3.</span> A deadlock just waiting to happen</div>
        <pre>public void transferFunds(Account from, Account to, Money amount) {
    synchronized (from) {
        synchronized (to) {
            from.decreaseBy(amount);
            to.increaseBy(amount);
        }
    }
}</pre>
</div>

(We'll ignore exceptional conditions such as insufficient funds. Or, if you like, assume that <code>decreaseBy()</code> throws an unchecked <code>InsufficientFundsException</code>.)

If it isn't obvious, the method is a service method that transfers funds from one account to another account. Before going further, take a look at the code and see if you can figure out how the deadlock can occur. Keep in mind what we said about cyclic dependencies.

<h4>Here's how a deadlock can occur</h4>

Recall from the discussion above that deadlocks involve multiple threads. Each one grabs a lock that one of the other threads needs, and needs a lock that one of the other threads has. Here, we have two <code>synchronized</code> blocks, and each entry point corresponds to a place where a thread will try to grab a lock, maybe successfully and maybe not.
    
With that information it's straightforward to construct the sequence of events that leads to the deadlock:

<ol>

<li>Thread T<sub>1</sub> enters the method, with the source account being account 4 and the destination account being account 14. T<sub>1</sub> is able to acquire the lock for account 4, but then there's a context switch before it can enter the second <code>synchronized</code> block.</li>

<li>Thread T<sub>2</sub> also enters the method, with the source account being account 14 and the destination account being account 4. T<sub>2</sub> is able to acquire the lock for account 14, but then gets blocked when trying to acquire the lock for account 4. That stands to reason, since T<sub>1</sub> has it. Context switch.</li>

<li>T<sub>1</sub> tries to grab the lock for account 14, but of course it can't since T<sub>2</sub> has it. Threads T<sub>1</sub> and T<sub>2</sub> are now deadlocked.</li>

</ol>

That's a fairly intricate bit of logic, and developers new to concurrent programming may wonder how in the heck they are supposed to anticipate problems like that. They may also wonder whether this series of events is so implausible that is just isn't worth worrying about. To answer the second concern first, deadlocks most certainly do occur in real code. Keep in mind that although the situations that produce them may be implausible, the problem is that CPU clock speeds are pretty incredible, and so things that seem implausible suddenly become just a matter of time.

As to how to defend against deadlocks, we'll look at that right now.

<h3>Solving deadlocks with globally-defined lock orderings</h3>

There are different ways to either minimize the likelihood of deadlocking or else just avoid it altogether. One best practice is to minimize the scope of your critical sections so as to decrease the time that threads hang onto locks, and hence to decrease the likelihood of a deadlock. That however doesn't solve the issue; deadlocks can still occur.

Another technique is to avoid the sort of nested locking that we saw in the example in the previous page. Instead of locking each account individually, we might define a single lock that any thread must acquire before doing a transfer. While this does indeed eliminate deadlocks, it does so at a very steep cost: acquiring the single lock becomes a bottleneck, and limits the parallelism we're trying to achieve in the first place. It does not scale.

A third technique is to use timeouts. Under this approach, if any thread is blocked for a sufficiently lengthy period of time, it just gives up.

Those are all legitimate techniques, but in this section we'll discuss global lock orders.

<h4>What is a globally-defined lock order?</h4>

Recall in our discussion of deadlocks that they arise from circular dependencies between the threads. Any given thread depends on a resource (a lock) that the previous thread in the ring holds. So one way to stamp out deadlocks is to define a global ordering on the locks so that such cycles can never arise. In other words, you define and publish rules around which locks to acquire before which other locks. This approach requires significant discipline from developers since there isn't any automatic way to enforce it. It is however extremely useful since it makes deadlocks impossible.

Let's see how we might apply a global lock order to the funds transfer example.

<h4>The funds transfer example reconsidered</h4>

At first glance you might think, "Wait a minute. The source account's lock is always grabbed before the destination account's lock, so how is it possible for a deadlock to occur?" But upon closer examination it's clear that that's not a correct analysis. Whatever global lock order we define needs to translate ultimately down to instances, not roles in a transaction. Sometimes account 4 will be the source account, and sometimes it will be the destination account. Similarly for account 14. Whatever global lock order we define needs to handle that.

One straightforward way to define a lock order for our current example would be to say that we always grab the lock for the account with the lower ID first. Listing 4 shows the new code:

<div>
    <div><span class="code-listing">Listing 4.</span> Applying a globally-defined lock order</div>
    <pre>public void transferFunds(Account from, Account to, Money amount) {
    long fromId = from.getId();
    long toId = to.getId();
    long lowId = (fromId &lt; toId ? fromId : toId);
    long highId = (fromId &lt; toId ? toId : fromId);
    synchronized (lowId) {
        synchronized (highId) {
            from.decreaseBy(amount);
            to.increaseBy(amount);
        }
    }
}</pre>
</div>

The code above is deadlock-free, because cyclic dependencies are impossible. Say thread T<sub>1</sub> wants to transfer funds from account 4 to account 14, and thread T<sub>2</sub> wants to transfer funds from account 14 to account 4. Both threads will try to grab the lock for account 4 before trying to grab the lock for account 14. Thus it's impossible to wait for the lock on account 4 while holding the lock for account 14, at least as long as all code observes the same globally-defined lock ordering that we've observed here. (Hence "global".)

<h3>Conclusion and recommended readings</h3>

In this series we looked at several key areas of concurrent programming that (in my opinion, anyway) all practicing software developers should understand. As regards concurrency, there is a gap between what typical developers <em>should</em> know and what they <em>actually</em> know, and that causes real-world production issues on a regular basis. I wrote this series to try to narrow that gap.

There are of course lots of good resources for concurrent programming. Here are some of my favorites:

<ul class="square">

<li><a href="http://www.infoq.com/presentations/goetz-concurrency-past-present">Concurrency: Past and Present</a>, by Brian Goetz: An entertaining presentation on concurrency. Goetz is a well-known authority on concurrent programming.</li>

<li><a href="http://jcip.net/">Java Concurrency in Practice</a>, by Brian Goetz. A really great and approachable book.</li>

<li><a href="http://www.amazon.com/Concurrent-Programming-Java-TM-Principles/dp/0201310090">Concurrent Programming in Java(TM): Design Principles and Pattern (2nd Edition)</a>, by Doug Lea. Another great book on concurrency; probably the standard. More academic than the Goetz book but still quite approachable.</li>

<li><a href="http://sunnyday.mit.edu/papers/therac.pdf">The Therac-25 Accidents</a>: A classic software engineering paper on how race conditions actually injured and killed people. The basic idea is that the Therac-25 machine, which was designed to administer radiation to cancer patients, was vulnerable to race conditions that occurred as the human operator became more experienced with operating the machine (and thus keyboarded commands more quickly). The race conditions led to massive radiation overdoses. Long but well worth the read.</li>

</ul>

<div class="outro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/10/15/concurrent-programming-for-practicing-software-engineers-part-3/">Part 3</a></span>
</div>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
