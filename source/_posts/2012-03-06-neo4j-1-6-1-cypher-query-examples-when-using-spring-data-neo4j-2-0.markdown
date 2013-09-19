---
layout: post
title: "Neo4j 1.6.1 Cypher query examples when using Spring Data Neo4j 2.0"
date: 2012-03-06 01:36:56
comments: true
categories: [Chapter 02 - Data, Chapter 11 - CMDB, Quick Tips]
---
This post shows how to perform various sample Cypher queries when using Neo4j 1.6.1 and Spring Data Neo4j (SDN) 2.0.

SDN 2.0 assumes Neo4j 1.6, and it imposes specific structures on databases that it creates. For example, SDN uses the <code>__type__</code> property on nodes to store the associated Java class, and it names its indexes using the Java class' simple name.

In the examples below, I'm using the Neo4j shell, though you can of course run Cypher queries outside of the shell (e.g., from within the app itself). See <a href="http://springinpractice.com/2012/02/12/working-with-the-neo4j-shell/" title="Working with the Neo4j shell">my earlier blog post</a> if you're interested in learning how to use the shell.

I'll probably add more examples to this post from time to time.

<h4>Return a single node with a given ID</h4>

<pre>neo4j-sh (Skybase,1)$ start n=node(1) return n
+------------------------------------------------------------------------------------------------------------------+
| n                                                                                                                |
+------------------------------------------------------------------------------------------------------------------+
| Node[1]{__type__-&gt;"org.skydingo.skybase.model.Application",name-&gt;"Skybase",shortDescription-&gt;"Cloud-based CMDB"} |
+------------------------------------------------------------------------------------------------------------------+</pre>

<h4>Return multiple nodes, looked up by ID</h4>

<pre>neo4j-sh (Skybase,1)$ start n=node(1,4,5) return n
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| n                                                                                                                                                                                  |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Node[1]{__type__-&gt;"org.skydingo.skybase.model.Application",name-&gt;"Skybase",shortDescription-&gt;"Cloud-based CMDB"}                                                                   |
| Node[4]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Domain",shortDescription-&gt;"Domain model",moduleId-&gt;"org.skydingo.skybase.domain",groupId-&gt;"org.skydingo.skybase"}     |
| Node[5]{moduleId-&gt;"org.skydingo.skybase.service",name-&gt;"Service",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Service module",__type__-&gt;"org.skydingo.skybase.model.Module"} |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+</pre>

<h4>Find all nodes of a given type</h4>

<pre>neo4j-sh[readonly] (Maven,3)$ start n=node:__types__(className="org.skydingo.skybase.model.Module") return n
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| n                                                                                                                                                                                  |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Node[3]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Maven",shortDescription-&gt;"Maven plugins",moduleId-&gt;"skybase-maven-plugin",groupId-&gt;"org.skydingo.skybase"}            |
| Node[4]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Domain",shortDescription-&gt;"Domain model",moduleId-&gt;"org.skydingo.skybase.domain",groupId-&gt;"org.skydingo.skybase"}     |
| Node[5]{moduleId-&gt;"org.skydingo.skybase.service",name-&gt;"Service",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Service module",__type__-&gt;"org.skydingo.skybase.model.Module"} |
| Node[7]{moduleId-&gt;"org.skydingo.skybase.client",name-&gt;"Client",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Client module",__type__-&gt;"org.skydingo.skybase.model.Module"}    |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+</pre>

<h4>Find all instances of a class having a certain property</h4>

Say we have a <code>org.skydingo.skybase.model.Module</code> class with a <code>groupId</code> property and a <code>moduleId</code> property, both annotated with SDN's <code>@Indexed</code> annotation. Then SDN will create an index for this class called <code>Module</code>. Here's how to find modules in a given group:

<pre>neo4j-sh (Skybase,1)$ start n=node:Module("groupId:org.skydingo.skybase") return n
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| n                                                                                                                                                                                  |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Node[3]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Maven",shortDescription-&gt;"Maven plugins",moduleId-&gt;"skybase-maven-plugin",groupId-&gt;"org.skydingo.skybase"}            |
| Node[4]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Domain",shortDescription-&gt;"Domain model",moduleId-&gt;"org.skydingo.skybase.domain",groupId-&gt;"org.skydingo.skybase"}     |
| Node[5]{moduleId-&gt;"org.skydingo.skybase.service",name-&gt;"Service",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Service module",__type__-&gt;"org.skydingo.skybase.model.Module"} |
| Node[7]{moduleId-&gt;"org.skydingo.skybase.client",name-&gt;"Client",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Client module",__type__-&gt;"org.skydingo.skybase.model.Module"}    |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+</pre>

Here's an alternative syntax for doing exactly the same thing:

<pre>neo4j-sh (Skybase,1)$ start n=node:Module(groupId = 'org.skydingo.skybase') return n
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| n                                                                                                                                                                                  |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Node[3]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Maven",shortDescription-&gt;"Maven plugins",moduleId-&gt;"skybase-maven-plugin",groupId-&gt;"org.skydingo.skybase"}            |
| Node[4]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Domain",shortDescription-&gt;"Domain model",moduleId-&gt;"org.skydingo.skybase.domain",groupId-&gt;"org.skydingo.skybase"}     |
| Node[5]{moduleId-&gt;"org.skydingo.skybase.service",name-&gt;"Service",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Service module",__type__-&gt;"org.skydingo.skybase.model.Module"} |
| Node[7]{moduleId-&gt;"org.skydingo.skybase.client",name-&gt;"Client",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Client module",__type__-&gt;"org.skydingo.skybase.model.Module"}    |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+</pre>

<h4>Find all nodes having a certain set of property values</h4>

<pre>neo4j-sh[readonly] (Maven,3)$ start n=node:Module("groupId:org.skydingo.skybase, moduleId:skybase-maven-plugin") return n  
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| n                                                                                                                                                                       |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Node[3]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Maven",shortDescription-&gt;"Maven plugins",moduleId-&gt;"skybase-maven-plugin",groupId-&gt;"org.skydingo.skybase"} |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+</pre>

<h4>Find all nodes in a specified relationship to a given node</h4>

<pre>neo4j-sh (Skybase,1)$ start app=node(1) match app-[:APPLICATION_MODULE]-&gt;module return module
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| module                                                                                                                                                                             |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Node[7]{moduleId-&gt;"org.skydingo.skybase.client",name-&gt;"Client",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Client module",__type__-&gt;"org.skydingo.skybase.model.Module"}    |
| Node[5]{moduleId-&gt;"org.skydingo.skybase.service",name-&gt;"Service",groupId-&gt;"org.skydingo.skybase",shortDescription-&gt;"Service module",__type__-&gt;"org.skydingo.skybase.model.Module"} |
| Node[4]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Domain",shortDescription-&gt;"Domain model",moduleId-&gt;"org.skydingo.skybase.domain",groupId-&gt;"org.skydingo.skybase"}     |
| Node[3]{__type__-&gt;"org.skydingo.skybase.model.Module",name-&gt;"Maven",shortDescription-&gt;"Maven plugins",moduleId-&gt;"skybase-maven-plugin",groupId-&gt;"org.skydingo.skybase"}            |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+</pre>

<h3>Reader-contributed</h3>

I'll just add queries here if people send them. It's useful to have examples of queries around.

<h4>Return a user's events (of specific types), ordered by date</h4>

<a href="https://twitter.com/#!/bytor99999">Mark Spritzler</a> sent me the following query, which he noted that <a href="https://twitter.com/#!/mesirii">Michael Hunger</a> mostly wrote:

<pre>START user=node({0})
MATCH user-[r]-event
WHERE type(r) = "ATTENDING"
OR type(r) = "INVITED"
OR type(r) = "HOSTING"
RETURN ID(event) as eventId, event.eventDate as eventDate, event.title as eventName, type(r) as eventUserType
ORDER BY event.eventDate desc</pre>
