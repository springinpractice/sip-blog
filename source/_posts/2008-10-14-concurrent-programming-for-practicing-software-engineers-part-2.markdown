---
layout: post
title: "Concurrent programming for practicing software engineers, part 2"
date: 2008-10-14 15:44:32
comments: true
categories: [Tutorials]
---
<div class="intro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/10/15/concurrent-programming-for-practicing-software-engineers-part-3/">Part 3</a></span>
</div>

This post is the second in a three-part series on concurrent programming. In the <a href="http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/">first post</a> we learned some basic concepts such as threads and serial vs. parallel execution. In this post we'll learn how parallel execution can create problems if we're not careful.

<h3>How parallelism gives rise to race conditions</h3>

Even among parallelizable problems, some are harder than others. The easy ones are the ones that don't involve threads sharing state with one another. For example, if you have five independent work queues, and the goal is simply to process all the tasks in all the queues, then there's nothing especially tough about this. You start five threads, run them against the five queues, and then move on once they're done.

The trickier problems are the ones that involve threads sharing state with one another. Let's see why.

<h4>Threads and shared state</h4>

Shared state presents a challenge in concurrent programming because threads can interfere with one another if their access to that shared state isn't properly coordinated. If you've ever tried to collaborate with other authors on a single document, you might have some experience with this problem. (Though Google Docs does a nice job of supporting multiple simultaneous authors.) You make some changes to the document, hoping that your coauthor didn't make his own changes to the part that you worked on. If he did, then there may be a mess to clean up.

The essence of the problem is that at least one guy is updating the document while somebody else is either reading it or writing it. We talked about the case where you have two authors writing to the document, but it can be one guy writing and someone else reading. If one person is in the middle of updating emergency information for hurricane evacuees and another person reads a partial update, the result could be dangerous.

Note that there's no problem if you have a bunch of people simply <em>reading</em> a document at the same time. The problem only comes up if at least one of them is <em>writing</em> to it.

The problem we've just described is called a <em>race condition</em>. The idea is that if threads aren't properly coordinated, then their concurrent work is vulnerable to problems associated with "unlucky timing". There can be situations in which one thread is trying to update shared state and one or more other threads are trying to access it (either read or write). In effect the threads "race" against one another and the outcome of their work is dependent on the nondeterministic outcome of their race.

Let's look at a code example.

<h4>An example of code that allows race conditions</h4>

Originally I was going to present the standard (and probably a bit tired) web page hit counter example. But then today I saw somebody's purportedly threadsafe class that is in fact susceptible to race conditions. The code was written by an experienced engineer. So let's use that example instead, shown in Listing 1 below, as it's instructive.

<div>
    <div>
        <span class="code-listing">Listing 1.</span> A class that's supposed to be threadsafe, but isn't
    </div>
    <pre>public class InMemoryUserDao implements UserDao {
    private Map&lt;String, User&gt; users;

    public InMemoryUserDao() {
        users = Collections.synchronizedMap(new HashMap&lt;String, User&gt;());
    }

    public boolean userExists(String username) {
        return users.containsKey(username);
    }

    public void createUser(User user) {
        String username = user.getUsername();
        if (userExists(username)) {
            throw new DuplicateUserException();
        }
        users.put(username, user);
    }

    public User findUser(String username) {
        User user = users.get(username);
        if (user == null) {
            throw new NoSuchUserException();
        }
        return user;
    }

    public void updateUser(User user) {
        String username = user.getUsername();
        if (!userExists(username)) {
            throw new NoSuchUserException();
        }
        users.put(username, user);
    }

    public void removeUser(String username) {
        if (!userExists(username)) {
            throw new NoSuchUserException();
        }
        users.remove(username);
    }
}</pre>
    </div>

What do you think? See anything? The engineer claims that the class is threadsafe because we've wrapped our hashmap with a synchronized instance.

<h4>Threadsafety problems with the code</h4>

It turns out that that's not correct. It's a common misconception that as long as you're dealing with a threadsafe collection (like a synchronized map), you've taken care of any threadsafety issues you might have otherwise had. We'll use this example to show why this belief is just plain wrong.

Before I expose the race conditions, let me state some plausible assumptions about the intended semantics for the class:

<ul class="square">
<li>Calling <code>createUser()</code> with an user whose username already exists in the map should generate a <code>DuplicateUserException</code>.</li>
<li>Calling <code>updateUser()</code> with an user whose username does not already exist should generate a <code>NoSuchUserException</code>.</li>
<li>Calling <code>removeUser()</code> with an user whose username does not already exist should generate a <code>NoSuchUserException</code>.</li>
</ul>

Now let's see how the implementation fails to support those semantics, and in particular how the implementation permits race conditions.

<b>createUser() is broken.</b> Here's a race that shows that <code>createUser()</code> does not have the intended semantics:

<ol>

<li>Thread T<sub>1</sub> enters <code>createUser()</code>, with argument user U<sub>1</sub> having username N. N is not already in the map.</li>

<li>Thread T<sub>1</sub> successfully makes it past the <code>if</code> test, but not yet to the line that puts N in the map.</li>

<li>Context switch to thread T<sub>2</sub>, which also enters <code>createUser()</code>, this time with argument user U<sub>2</sub> having the same username N that U<sub>1</sub> has. (You can imagine, for example, two users registering with the same username at the same time.) N is still not in the map, because thread T<sub>1</sub> hasn't put it there yet.</li>

<li>Thread T<sub>2</sub> completes its call to <code>createUser()</code> and exits. User U<sub>2</sub> and username N are now in the map.</li>

<li>Thread T<sub>1</sub> completes the method and executes. User U<sub>1</sub> has been placed into the map under the key N.</li>

</ol>

Poof! User U<sub>2</sub> simply disappears from the map, having been overwritten by user U<sub>1</sub>. T<sub>1</sub> never encounters the <code>DuplicateUserException</code> it's supposed to encounter.

<b>updateUser() is broken.</b> This one is a little more subtle than the <code>createUser()</code> case. Here's the race:

<ol>
<li>Thread T<sub>1</sub> enters <code>updateUser()</code> with user U having name N. The user already exists in the map, so T<sub>1</sub> passes the <code>if</code> test successfully. It hasn't reached the <code>put()</code> part yet.</li>
<li>Context switch to thread T<sub>2</sub>, which enters and exits <code>removeUser()</code> with username N (the same one that T<sub>1</sub> was using). The user U is removed from the map.</li>
<li>Thread T<sub>1</sub> completes, placing U back in the map under key N. User U has essentially been reinstated.</li>
</ol>

The problem here is that the <code>removeUser()</code> method completed successfully, there was no <code>createUser()</code> call, and yet there's still a user sitting in the map with no error having been thrown. Given the semantics described above, that should not be possible. The only two possible outcomes ought to be: (1) the update occurs before the remove, which would mean that when the two threads complete, U is no longer in the map; or (2) the remove occurs before the update, which would mean that U should be gone, and the update should have generated an exception. In either case, U should not be in the map when the two threads complete, and yet there it is. So if T<sub>2</sub> belongs to an admin who thought he was removing a troublesome user, guess again.

<b>removeUser() is broken.</b> We've already seen in the race above that there are situations under which <code>removeUser()</code> would be expected to remove a user but does not actually do that. Here's another race for you, probably less significant, but still a bug:

<ol>
<li>Thread T<sub>1</sub> enters <code>removeUser()</code> with existing username N, and makes it past the <code>if</code> test.</li>
<li>Context switch to thread T<sub>2</sub>, which enters and exits <code>removeUser()</code> with the same username N.</li>
<li>Thread T<sub>1</sub> completes and exits the method.</li>
</ol>

In this case we have two methods that called the <code>removeUser()</code> method with the same username, but neither thread encountered a <code>NoSuchUserException</code>. That is not compatible with the semantics of the class.

Now that we've seen some problems, let's try to understand what's happening here.

<h4>What the three broken methods have in common</h4>

One thing that all three cases have in common is that they follow a "check-then-act" pattern, where both the "check" and "act" parts reference shared data (in this case, the hashmap). The <code>createUser()</code> method checks whether the username already exists, and if not, adds the user to the map. The <code>updateUser()</code> method checks whether the username already exists, and if so, updates the user. <code>removeUser()</code> checks whether the username exists, and if so, deletes the user.

Note that the <code>findUser()</code> method does <em>not</em> follow the same pattern. It does a single <code>get()</code> against the map, and then does a check that checks the returned user rather than checking something against the map. Though it looks like almost the same thing, it is really a completely different situation because we're performing only one action against the shared data instead of multiple actions.

It turns out that the difference between <code>createUser()</code>, <code>updateUser()</code> and <code>removeUser()</code>, on the one hand, and the <code>findUser()</code> method, on the other, is important from a threadsafety point of view. Essentially what's happening is that the <code>findUser()</code> method is <em>atomic</em> (with a certain qualification, which we'll discuss in the next section), meaning that it executes as a single action. The <code>createUser()</code>, <code>updateUser()</code> and <code>removeUser()</code> methods, on the other hand, are not atomic. The various check and act operations can be interleaved in arbitrary ways across different threads. Intuitively what that means is that checks, which are supposed to provide safety, can become invalid because other actions can be scheduled in between any given check-act pair. The safety checks are invalidated and the class behaves in strange ways (especially as load increases, which is usually the most inopportune time for such issues to arise).

Now let's look at how we can solve race conditions such as the ones we've been considering.

<h3>Solving race conditions with synchronization</h3>

The key to avoiding race conditions is to coordinate the multiple threads when accessing shared data. There are different ways to do this, but the simplest is the one your mom taught you when you were a kid: take turns.

<h4>Taking turns with mutexes</h4>

Here's how you take turns in software. Suppose that you have some shared data that you want to protect, such as a list of all the posts in a given web-based discussion forum, or a web page hit counter. You have various blocks of code throughout your application that access that data, both reading it and writing it. These blocks of code don't have to be in the same class (though they often are), but they're all accessing the same data.

(Keep in mind that when we describe the data as shared data, we mean that it's shared by multiple threads, not that multiple blocks of code access it. If multiple blocks of code access the data but only one thread ever does, then it's not shared data.)

The trick is to protect the code so that only one thread can enter it at a time. I don't mean that only one thread can enter one of the code blocks at a time. I mean that if you consider the whole set of code blocks as a single group, only one thread can enter that group at a time. So if one thread enters one of the blocks, then all other threads need to be kept out of all other blocks until that first thread is done.

The way we enforce this behavior is to associate with each block of code you want to protect something called a mutual exclusion lock, or <em>mutex</em>. People often just call it a lock as well. You associate a lock with any given piece of data you want to protect, and all the code blocks that access that data use that same lock. A thread is not allowed to enter the code block unless it acquires the lock. So whenever thread T<sub>1</sub> wants to enter, it attempts to grab the lock. If the lock is available, then T<sub>1</sub> takes it and other threads have to wait until T<sub>1</sub> is done. If instead some other thread T<sub>2</sub> has the lock, then T<sub>1</sub> has to wait until T<sub>2</sub> is done.

When a thread leaves the block of code protected by the mutex, known as the <em>critical section</em>, it releases the lock.
    
Incidentally, I always thought it was kind of funny we talk about grabbing locks. It seems like we should say we grab a key so we can get past the lock. But that's not how people talk about it.

We now introduce the principal way of implementing a mutex in Java, the <code>synchronized</code> keyword.

<h4>The <code>synchronized</code> keyword</h4>

In Java the primary mechanism for enforcing turn-taking across threads is the <code>synchronized</code> keyword. You basically put the code you want to protect inside a <code>synchronized</code> block, specifying a lock object (or more exactly, specifying an object whose <em>monitor</em> will be used as a lock&mdash;in Java, each object has a "monitor" that can be used as a mutex). If you want to synchronize access to the whole method, you just use the <code>synchronized</code> keyword on the method itself. In this latter case, the locking object is the one on which the method is being called.
    
Here's an example of a synchronized block:

<pre>synchronized (myList) {
    int lastIndex = myList.size() - 1;
    myList.remove(lastIndex);
}</pre>

Here, we're using the <code>myList</code> instance as the lock. Anytime a thread wants to enter this block of code, or any other block of code protected by the same lock, it needs to grab the lock on the <code>myList</code> instance first. And it releases the lock upon exiting the <code>synchronized</code> block (i.e., the critical section).
    
Here's an example of a synchronized method:

<pre>public class Order {

    ...

    public synchronized void removeLastLineItem() {
        int lastIndex = myList.size() - 1;
        myList.remove(lastIndex);
    }
}</pre>

In this case, a thread wanting to enter the <code>removeLastLineItem()</code> method would need to grab the lock on the relevant <code>Order</code> instance first, and would release the lock after exiting the method.

Armed with our understanding of mutexes and the <code>synchronized</code> keyword, let's revisit our example from the previous page.

<h4>Our example, revisited: making the code threadsafe</h4>

Listing 2 shows how to apply thread synchronization to make the class threadsafe.

<div>
    <div><span class="code-listing">Listing 2.</span> A threadsafe version of our DAO</div>
    <pre>public final class InMemoryUserDao implements UserDao {
    private Map&lt;String, User&gt; users;

    public InMemoryUserDao() {
        users = Collections.synchronizedMap(new HashMap&lt;String, User&gt;());
    }

    public boolean userExists(String username) {
        return users.containsKey(username);
    }

    public void createUser(User user) {
        String username = user.getUsername();
        synchronized (users) {
            if (userExists(username)) {
                throw new DuplicateUserException();
            }
            users.put(username, user);
        }
    }

    public User findUser(String username) {
        User user = users.get(username);
        if (user == null) {
            throw new NoSuchUserException();
        }
        return user;
    }

    public void updateUser(User user) {
        String username = user.getUsername();
        synchronized (users) {
            if (!userExists(username)) {
                throw new NoSuchUserException();
            }
            users.put(username, user);
        }
    }

    public void removeUser(String username) {
        synchronized (users) {
            if (!userExists(username)) {
                throw new NoSuchUserException();
            }
            users.remove(username);
        }
    }
}</pre>
    </div>
    
In Listing 2 we've addressed the three issues that we raised earlier. First note that the <code>users</code> instance is entirely encapsulated by our <code>InMemoryUserDao</code> class (I've made the class <code>final</code> to prevent inheritance from breaking our encapsulation here), so that means it is entirely within our power to protect every possible access to the <code>users</code> instance. (Hm, that's probably a good interview question: "Discuss the relationship between encapsulation and threadsafety.")

Next note that we've addressed the general problem that we saw by making every "check-then-act" occurrence atomic. Now the code is such that if thread T checks something (such as checking whether a username exists), we can be sure that the answer doesn't change before the action part, because no other thread is able to access the <code>users</code> instance at all until T is done with it.

<h4>Internal vs. external synchronization</h4>

There's one detail to discuss. Take a look at the <code>userExists()</code> and <code>findUser()</code> methods. In both cases we access the <code>users</code> instance, but there's no <code>synchronized</code> block. The reason this isn't a problem is that both of the following are true:
    
<ol>
<li>we're calling only one method on the <code>users</code> map, and</li>
<li>the map is internally synchronized since we created the map using the <code>Collections.synchronizedMap(...)</code> wrapper.</li>
</ol>

If either of those two conditions had failed, we'd need to put the calls in a <code>synchronized</code> block. It's useful to discuss these a little more carefully.

Let's take (1) first. Note that even though we don't have an explicit <code>synchronized</code> block, each individual method call we make against <code>users</code> is threadsafe. That's because the <code>users</code> map is internally synchronized on the monitor for the <code>users</code> instance itself. That is, the map handles synchronization for individual method calls so we don't have to. If we're just going to call a single method on an internally synchronized method, we're usually OK to do so.
    
The place where we have to pay attention to threadsafety is where we call multiple methods on the <code>users</code> instance. There, the internal synchronization doesn't help us at all because that synchronization doesn't prevent the thread from releasing the lock in between the multiple method calls. And when it releases the lock, another thread can come in and change the data. If your code logic assumes that the multiple method calls behave as an atomic unit, you have a problem. Therefore you must provide your own synchronization on the relevant lock when you use check-then-act and similar patterns.

It is a common error to assume that just because the object is internally synchronized, you don't have to worry about threadsafety anymore. That is simply not true, as we've just explained.
    
Now take (2). Clearly if the map didn't have any internal synchronization, then we'd need to provide it ourselves, even for single method calls against the map. Otherwise threads could get in and muck things up for other threads. This is true even in many cases where it <em>looks</em> like individual threads are performing atomic actions. For example, if you have some threads making <code>get()</code> calls against a <code>HashMap</code>, and another set of threads making <code>put()</code> calls against the same <code>HashMap</code>, you may think that they are all performing atomic operations and therefore it's OK to do this. But that is not correct. The problem is that they only <em>look</em> like atomic operations; behind the scenes, they're impacting the size of a shared backing array, which in turn impacts how hashcodes are mapped to buckets, which impacts the <code>get()</code> and <code>put()</code> calls themselves. If the class had internal synchronization then <code>get()</code> and <code>put()</code> would both be atomic from the perspective of external code, but without that, they aren't atomic at all.
    
That was your crash course on solving race conditions with thread synchronization. It's not a simple topic, but you should now have the basics. But even though thread synchronization helps solve one problem, unfortunately it creates another. The problem is that in forcing threads to stop and wait their turn, you create the potential for the situation in which threads might wait indefinitely before they can move on. This is called a deadlock, and we'll turn to it in the next post in the series.

<div class="outro">
<span class="icon stickyNote">This post is part of a three-part series: <a href="http://springinpractice.com/2008/10/13/concurrent-programming-for-practicing-software-engineers-part-1/">Part 1</a> | <a href="http://springinpractice.com/2008/10/14/concurrent-programming-for-practicing-software-engineers-part-2/">Part 2</a> | <a href="http://springinpractice.com/2008/10/15/concurrent-programming-for-practicing-software-engineers-part-3/">Part 3</a></span>
</div>

<div class="endnote">Post migrated from my Wheeler Software site.</div>
