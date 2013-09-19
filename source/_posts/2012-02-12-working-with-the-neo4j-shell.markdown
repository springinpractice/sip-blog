---
link: http://springinpractice.com/2012/02/12/working-with-the-neo4j-shell/
layout: post
title: Working with the Neo4j shell
date: 2012-02-12 14:06:54
comments: true
categories: [Chapter 02 - Data, Chapter 11 - CMDB, Quick Tips, Tutorials]
---
In this post we're going to learn how to work with a local Neo4j database using the <a href="http://neo4j.org/" title="Neo4j">Neo4j</a> shell. This isn't really a Spring post, but <a href="http://www.springsource.org/spring-data/neo4j" title="Spring Data Neo4j">Spring Data Neo4j</a> users will probably find it useful.

<strong>Set environment variable (optional).</strong> First, set the <code>NEO4J_HOME</code> environment variable to point to the top-level Neo4j directory, and put the <code>bin</code> directory on your path.

<strong>Start the Neo4j shell.</strong> Type

<code>neo4j-shell -path path/to/neo4j-db</code>

with the actual path to your database substituted in. If you want to run in read-only mode, you can do this instead:

<code>neo4j-shell -readonly -path path/to/neo4j-db</code>

If everything went well, you should see something that looks like this:
<pre>/lib/neo4j-community-1.5/bin$ neo4j-shell -path ~/projects/skydingo/skybase/neo4j/db
NOTE: Local Neo4j graph database service at '/Users/williewheeler/projects/skydingo/skybase/neo4j/db'
Welcome to the Neo4j Shell! Enter 'help' for a list of commands

Welcome to the Neo4j Shell! Enter 'help' for a list of commands

neo4j-sh (0)$</pre>
I'm not sure why it welcomes me twice (friendly shell, I suppose), but that's what it does.

<strong>Try some commands.</strong> Now we're in the shell. The command set is bash-like, which is kind of nice if you're already familiar with bash. Let's play around with some commands.

First, we'll set the current node to node 22:

<pre>neo4j-sh (0)$ cd -a 22
neo4j-sh (willie,22)$</pre>

The <code>-a</code> flag stands for "absolute path", and it just means that I can navigate to any node at all in the graph, instead of being limited to adjacent nodes.

Now let's see what node 22 looks like:

<pre>neo4j-sh (willie,22)$ ls
*__type__   =[org.skydingo.skybase.model.Person]
*email      =[willie@example.com]
*firstName  =[Willie]
*gitHubUser =[williewheeler]
*lastName   =[Wheeler]
*title      =[Lead developer]
*username   =[willie]
*workPhone  =[999-111-2222]
neo4j-sh (willie,22)$</pre>

Let's change the work phone:

<pre>neo4j-sh (willie,22)$ set workPhone "999-867-5309"
neo4j-sh (willie,22)$ ls
*__type__   =[org.skydingo.skybase.model.Person]
*email      =[willie@example.com]
*firstName  =[Willie]
*gitHubUser =[williewheeler]
*lastName   =[Wheeler]
*title      =[Lead developer]
*username   =[willie]
*workPhone  =[999-867-5309]
neo4j-sh (willie,22)$ </pre>

You can tell that I created the node above using Spring Data Neo4j since it has the <code>__type__</code> property that Spring Data Neo4j uses.

Now let's create a new node. With Neo4j we create new nodes by relating them to existing nodes. Witness:

<pre>neo4j-sh (willie,22)$ mkrel -t HAS_HOBBY -d OUTGOING -c
neo4j-sh (willie,22)$ ls    
*__type__   =[org.skydingo.skybase.model.Person]
*email      =[willie@example.com]
*firstName  =[Willie]
*gitHubUser =[williewheeler]
*lastName   =[Wheeler]
*title      =[Lead developer]
*username   =[willie]
*workPhone  =[999-867-5309]
(me) --[HAS_HOBBY]-&gt; (48)
neo4j-sh (willie,22)$</pre>

Notice that there's now a node 48. Let's navigate to that node and give it a title:

<pre>neo4j-sh (willie,22)$ cd 48
neo4j-sh (48)$ ls
(me) &lt;-[HAS_HOBBY]-- (willie,22)
neo4j-sh (48)$ set name &quot;Playing guitar&quot;
neo4j-sh (Playing guitar,48)$ ls
*name =[Playing guitar]
(me) &lt;-[HAS_HOBBY]-- (willie,22)
neo4j-sh (Playing guitar,48)$</pre>

Alright, that was cool, but I don't want that node anymore. We're going to delete both the node and the relationship we created:

<pre>neo4j-sh (Playing guitar,48)$ rmnode 48
(Playing guitar,48) cannot be deleted because it still has relationships. Use -f to force deletion of its relationships
neo4j-sh (Playing guitar,48)$ rmnode -f 48
Relationship [HAS_HOBBY,26] deleted
neo4j-sh (?)$ cd -a 22
neo4j-sh (willie,22)$ ls
*__type__   =[org.skydingo.skybase.model.Person]
*email      =[willie@example.com]
*firstName  =[Willie]
*gitHubUser =[williewheeler]
*lastName   =[Wheeler]
*title      =[Lead developer]
*username   =[willie]
*workPhone  =[999-867-5309]
neo4j-sh (willie,22)$</pre>

We can do Cypher queries too. Here's a pretty basic one:

<pre>neo4j-sh (willie,22)$ start n=node(2) return n
+------------------------------------------------------------------------+
| n                                                                      |
+------------------------------------------------------------------------+
| Node[2]{name-&gt;"US West",__type__-&gt;"org.skydingo.skybase.model.Region"} |
+------------------------------------------------------------------------+
1 rows, 0 ms
neo4j-sh (willie,22)$</pre>

OK, it's quitting time:

<pre>neo4j-sh (willie,22)$ quit
/lib/neo4j-community-1.5/bin$</pre>

There are other commands too, and for those, see the <a href="http://docs.neo4j.org/" title="Neo4j Reference Manual">Neo4j reference manual</a>. But now you should be able to perform basic operations inside the shell.

Have fun!
