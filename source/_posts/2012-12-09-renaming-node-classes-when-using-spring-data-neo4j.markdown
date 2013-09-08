---
layout: post
title: "Renaming node classes when using Spring Data Neo4j"
date: 2012-12-09 15:33
comments: true
categories: [Quick Tips, Chapter 11 - CMDB]
---
This is one of those posts where I'm just jotting down some notes for my own future use, but someone else may find this useful.

I'm going to show how to rename a node entity class when using Spring Data Neo4j. I'm talking about the fully-qualified classname here, so it applies when we want to rename a package too.

First, suppose my old classname is <code>org.zkybase.cmdb.api.domain.ApplicationEntity</code>. If I go into the Neo4j shell, I can see that I have a couple of nodes of this type:

    neo4j-sh (0)$ start n=node:__types__(className="org.zkybase.cmdb.api.domain.ApplicationEntity") return n
    +-----------------------------------------------------------------------------------------------------+
    | n                                                                                                   |
    +-----------------------------------------------------------------------------------------------------+
    | Node[36]{__type__->"org.zkybase.cmdb.api.domain.ApplicationEntity",name->"Zkybase"}                 |
    | Node[49]{name->"Spring in Practice Blog",__type__->"org.zkybase.cmdb.api.domain.ApplicationEntity"} |
    +-----------------------------------------------------------------------------------------------------+
    2 rows, 1 ms

From the above, it <i>looks like</i> you can just go to the nodes themselves and change their <code>__type__</code> fields:

    neo4j-sh (0)$ cd -a 36
    neo4j-sh (Zkybase,36)$ ls
    *__type__ =[org.zkybase.cmdb.api.domain.ApplicationEntity]
    *name     =[Zkybase]
    neo4j-sh (Zkybase,36)$ set __type__ "org.zkybase.api.domain.entity.ApplicationEntity"
    neo4j-sh (Zkybase,36)$ ls
    *__type__ =[org.zkybase.api.domain.entity.ApplicationEntity]
    *name     =[Zkybase]

But then when you try to find the node using the query, it doesn't show up.

    neo4j-sh (Zkybase,36)$ start n=node:__types__(className="org.zkybase.api.domain.entity.ApplicationEntity") return n
    +---+
    | n |
    +---+
    +---+
    0 rows, 1 ms

Moreover, when you re-run the original query, the node whose <code>__type__</code> we changed still shows up.

The problem is that we need to reindex the nodes. Spring Data Neo4j uses an index called <code>__types__</code>, and we need to replace the old index entries with some new ones.

Let's see what's under the old classname, using <code>index -g</code> to get the relevant nodes from the <code>__types__</code> index:

    neo4j-sh (Zkybase,36)$ index -g __types__ className "org.zkybase.cmdb.api.domain.ApplicationEntity"
    (me)
    
    (Spring in Practice Blog,49)

And under the new classname:

    neo4j-sh (Zkybase,36)$ index -g __types__ className "org.zkybase.api.domain.entity.ApplicationEntity"
    neo4j-sh (Zkybase,36)$ 

We can fix that using <code>index -i</code>, which indexes the current entity:

    neo4j-sh (Zkybase,36)$ index -i __types__ className "org.zkybase.api.domain.entity.ApplicationEntity"
    neo4j-sh (Zkybase,36)$ cd -a 49
    neo4j-sh (Spring in Practice Blog,49)$ index -i __types__ className "org.zkybase.api.domain.entity.ApplicationEntity"
    neo4j-sh (Spring in Practice Blog,49)$ cd -a 36
    neo4j-sh (Zkybase,36)$ index -g __types__ className "org.zkybase.api.domain.entity.ApplicationEntity"
    (me)
    
    (Spring in Practice Blog,49)

We still need to clean up the old entries, though, because they're still there. We use <code>index -r</code> to remove the index entry for the current node:

    neo4j-sh (Zkybase,36)$ index -r __types__ className "org.zkybase.cmdb.api.domain.ApplicationEntity"
    neo4j-sh (Zkybase,36)$ cd -a 49
    neo4j-sh (Spring in Practice Blog,49)$ index -r __types__ className "org.zkybase.cmdb.api.domain.ApplicationEntity"
    neo4j-sh (Spring in Practice Blog,49)$ index -g __types__ className "org.zkybase.cmdb.api.domain.ApplicationEntity"
    neo4j-sh (Spring in Practice Blog,49)$ 

That's it. This was the result of about 15 minutes of investigation, so there's a good chance there's more going on than what I've described. Let me know and I'll update the post accordingly.
