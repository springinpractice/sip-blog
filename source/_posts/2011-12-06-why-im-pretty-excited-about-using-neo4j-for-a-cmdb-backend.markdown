---
layout: post
title: "Why I'm pretty excited about using Neo4j for a CMDB backend"
date: 2011-12-06 03:14:44
comments: true
categories: [Architecture, Chapter 11 - CMDB]
---
![deathburro](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-06-why-im-pretty-excited-about-using-neo4j-for-a-cmdb-backend/deathburro-200x300.jpg)

[Zkybase](https://github.com/williewheeler/zkybase) is my first <em>open source</em> configuration management database (CMDB) effort, but it's not the first time I've built a CMDB. At work a bunch of us built--and continue to build--a proprietary, internal system CMDB called deathBURRITO as part of our deployment automation effort. We built deathBURRITO using Java, Spring, Hibernate and MySQL. deathBURRITO even has a <a href="http://theksmith.com/tagged/deathburro/">robotic donkey</a> (really) whose purpose we haven't quite yet identified.

So far deathBURRITO has worked out well for us. Some of its features--namely, those that directly support deployment automation--have proven more useful than others. But the general consensus seems to be that deathBURRITO addresses an important configuration management (CM) gap, where previously we were "managing" CM data on a department wiki, in spreadsheets, in XML files and in Visio diagrams. While there's more work to do, what we've done so far has been reasonably right-headed, and we've been able to evolve it as our needs have evolved.

That's not to say that there's nothing I would change. I think there's an opportunity to do something better on the backend. That was indeed the impetus for Skybase.

<h3>Revisiting the backend with Spring Data</h3>

Because CM involves a lot of different kinds of entities (e.g., regions, data centers, environments, farms, instance types, machine images, instances, applications, middleware, packages, deployments, EC2 security groups, key pairs--the list goes on and on), it was helpful to build out a bit of framework to handle various cross-cutting concerns more or less automatically, such as standard views (list view, details view, form views), web controllers (CRUD ops, standard queries), DAOs (again, CRUD ops and queries), generic domain object capabilities, security and more. And I think it's fair to say that this helped, though perhaps not quite as much as I would have liked (at least not yet). The fact remains that anytime we want to add a new entity to the system, we still have to do a fair amount of work making each layer of the framework do what it's supposed to do.

My experience has been that the data persistence layer is the one that's most challenging to change. Besides the actual schema changes, we have to write data migration scripts, we have to make corresponding changes to our integration test data scripts, we have to make sure Hibernate's eager- and lazy-loading are doing the right things, sometimes we have to change the domain object APIs and associated Hibernate queries, etc. Certainly doable, but there's generally a good deal of planning, discussion and testing involved.

So I started looking for some ways to simplify the backend.

Recently I started goofing around with <a href="http://www.springsource.org/spring-data">Spring Data</a>, and in particular, <a href="http://www.springsource.org/spring-data/jpa">Spring Data JPA</a> and <a href="http://www.springsource.org/spring-data/mongodb">Spring Data MongoDB</a>. For those who haven't used Spring Data, it offers some nice features that I wanted to try out:

<ul class="square">
	<li>One of the features is that it's able to generate DAO implementations automagically using Java's dynamic proxy mechanism. Spring Data calls these DAOs "repositories", which is fine with me. Basically what you do as a developer is you write an interface that extends a type-specific Repository interface, such as a JpaRepository or a MongoRepository. You don't have to declare CRUD operations because the Repository interface already declares the ones you'd typically want (e.g., save(), findAll(), findById(), delete(), deleteAll(), exists(), count(), etc.).</li>
	<li>And if you have custom queries, just declare them on the interface using some method naming conventions and Spring Data can figure out how to generate an implementation for you.</li>
	<li>In some cases, Spring Data handles mapping for you. In the case of Spring Data JPA you still have to make sure you have the right JPA annotations in place, and that's not too terribly bad. But for MongoDB, since the MongoDB backend is just BSON (binary JSON), the mapping from objects to MongoDB is straightforward and so Spring Data handles that very well, without requiring a bunch of annotations.</li>
</ul>
"Game changer" might be too strong for the features I've just described, but man, are they useful. Besides making it easier and faster to implement repos, Spring Data allows me to get rid of a lot of boilerplate code (lots of DAO implementations) and even some Spring Data-like DAO framework code that I usually write. (You know, define a generic DAO interface, a generic abstract DAO class, etc.) My days of building DAOs directly against Hibernate are probably over.

But that's not the only backend change I'm looking at.

<h3>Revisiting the backend, part 2: Neo4j + Spring Data Neo4j</h3>

![Example of a graph in Neo4j](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-06-why-im-pretty-excited-about-using-neo4j-for-a-cmdb-backend/neo4j-graph.png)

Since Spring Data tends to gravitate toward the NoSQL stores, I finally got around to reading up on Neo4j and some other stuff that I probably ought to have read up on a long time ago. Better late than never I guess. It struck me that Neo4j could be a very interesting way to implement a CM backend, and that Spring Data Neo4j could help me keep the DAO layer thin. Here's the thinking:

<ul class="square">
	<li><strong>There are many entities and relationships.</strong> There are lots and lots of entities and relationships in a CMDB. There are different ways of grouping things, like grouping which assets form a stack, which stacks to deploy to which instances, which instances are in which farms, how to define shards and how to define failover farms, grouping features into apps, endpoints into services, etc. We have to associate SLAs and OLAs with app features and service endpoints, we have to define dependencies between components, and so on. There's a ton of stuff. It would eliminate a major chunk of work if we could get away with defining the schema one time (say, in the app itself) instead of in both the app and the database.</li>
	<li><strong>We need schema agility to experiment with different CMDB approaches.</strong> Along similar lines, there are some strong forces that push for schema agility. Most fundamentally, the right approach to designing and implementing a CMDB isn't necessarily a solved problem. Some tools involve running network discovery agents that collect up millions of configuration items (CIs) from the environment. Other tools focus more on collecting the <em>right</em> data rather than collecting everything. Some tools have more of a traditional enterprise data center customer in mind, where you're capturing everything from bare metal to the apps to the ITIL services based on the apps. Other tools are more aligned with cloud infrastructures, starting from virtualized infrastructure and working up, leaving everything under that virtualization layer for the cloud provider to worry about. Some tools treat CIs as supporting configuration management but not application performance management (APM); other tools try to single-source their CM and APM data. Some organizations want to centralize CIs in a single database and other organizations pursue a more federated model. With such a panoply of approaches, it's no surprise that schema changes occur.</li>
	<li><strong>We need schema agility to accommodate continuing innovations in infrastructure.</strong> Another schema agility driver is the fact that the way we do infrastructure is rapidly evolving. Infrastructure is steadily moving to the cloud, virtualization is commonplace and new automation and monitoring tools are appearing all the time. The constant stream of innovation demands the ability to adjust quickly, and schema flexibility is a big part of that.</li>
	<li><strong>We need schema flexibility to accommodate the needs of different organizations.</strong> Besides schema agility, a CMDB needs to offer a certain level of flexibility to support the needs of different customers. One org may have everything in the cloud, where another one is just getting its feet wet. One org may have two environments whereas another has seven. Different orgs have different roles, test processes, deployment pipelines and so forth. Any CMDB that hopes to support more than just a single customer needs to support some level of flexibility.</li>
	<li><strong>But we still need structure.</strong> One of the more powerful benefits of driving deployment automation from a CMDB is that you can control drift in the target environments by controlling the data in your CMDB. And defining a schema around that is a great way to do so. If, for example, you expect every instance in a given farm to have the same OS, then your schema should attach the OS to the farm itself, not to the instance. (The instances inherit the OS from the farm.) Or maybe it's not just the OS--you want a certain instance type (in terms of virtual hardware profile), certain machine image, certain middleware stack, certain security configuration, etc. Great: bundle those together in a single server build object and then associate that with the farm. This constrains the instances in the farm to have the same server build. (See <a title="Flexibility" href="http://skydingo.com/blog/?p=56">this post</a> for a more detailed discussion.) While Neo4j is schema-free, Spring Data Neo4j gives us a way to impose schema from the app layer.</li>
	<li><strong>A schemaless backend makes zero-downtime deployments easier.</strong> If you have any apps with hardcore availability requirements, then it goes without saying that you have to have significant automation in place to support that, both on the deployment side and on the incident response side. Since the CMDB is the single source of truth driving that automation, it follows that the CMDB itself must have hardcore availability requirements. Having a schemaless database ought to make it easier to perform zero-downtime deployments of the CMDB itself.</li>
	<li><strong>We want to support intuitive querying.</strong> Once you bring a bunch of CIs together in a CMDB, there are all kinds of queries that pop up. This could be anything from using the dependency structure to diagnose an incident, assess the impact of a change, or sequence project deliverables (e.g. make system components highly available depending on where they sit in the system dependency graph); using org structures to determine support escalation paths and horizontal communications channels; SLA reporting by group, manager, application category; and so forth. Intuitive query languages for graph databases--and in particular, the Cypher query language for Neo4j--appear to be especially well-suited for the broad diversity of queries we expect to see.</li>
</ul>

Remember how I mentioned that Spring Data generates repository implementations automatically based on interfaces? Here's what that looks like with Spring Data Neo4j:

    package org.skydingo.skybase.repository;
    
    import org.skydingo.skybase.model.Person;
    import org.skydingo.skybase.model.Project;
    import org.springframework.data.neo4j.annotation.Query;
    import org.springframework.data.neo4j.repository.GraphRepository;
    
    public interface ProjectRepository extends GraphRepository<Project> {
        Project findProjectByKey(String key);
        Project findProjectByName(String name);
    
        @Query("start person=node({0}) match person-->project return project")
        Iterable<Project> findProjectsByPerson(Person person);
    }

Let me repeat that I <em>don't</em> have to write the repository implementation myself. <code>GraphRepository</code> comes with various CRUD operations. Methods like <code>findProjectByKey()</code> and <code>findProjectByName()</code> obey a naming convention that allows Spring Data to produce the backing query automatically. And in the <code>findProjectsByPerson()</code> case, I provided a query using Neo4j's Cypher query language (it uses ASCII art to define queries--how ridiculously cool is that?).

<h3>Exploring the ideas above with Skybase</h3>

![Zkybase dashboard](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-06-why-im-pretty-excited-about-using-neo4j-for-a-cmdb-backend/dashboard1.png)

The point of Skybase is to see whether we can build a better mousetrap based on the ideas above. I'm using Neo4j and Spring Data Neo4j to build it out. I haven't decided yet whether Skybase will focus on the CMDB piece or whether it's a frontend to configuration management more generally (delegating on the backend to something like <a href="http://www.opscode.com/chef/">Chef</a> or <a href="http://puppetlabs.com/">Puppet</a>, say), but the CMDB will certainly be in there. That will include a representation of the as-is (current) configuration as well as representations for desired configurations as might be defined during a deployment planning activity.

So far I'm finding it a lot easier to work with the graph database than with a relational database, and I'm finding Spring Data Neo4j to be a big help in terms of repository building and defining app-level schemas. The code is a lot smaller than it was when I did this with a relational database. But it's still early days, so the jury is out.

Watch this space for further developments.
