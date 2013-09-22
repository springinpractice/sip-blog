---
layout: post
title: "Domain modeling with Spring Data Neo4j"
date: 2011-12-17 23:26:41
comments: true
categories: [Chapter 02 - Data, Chapter 11 - CMDB, Tutorials]
---
Hi all, Willie here. Last time I told you that I'm building the [Zkybase](https://github.com/williewheeler/zkybase) CMDB using <a title="Neo4j" href="http://neo4j.org/">Neo4j</a> and <a title="Spring Data Neo4j" href="http://www.springsource.org/spring-data/neo4j">Spring Data Neo4j</a>, and I was excited to get a lot of positive feedback about that. I showed a little code but not that much. In this post I'll show you how I'm building out the person <a title="Configuration item" href="http://en.wikipedia.org/wiki/Configuration_item">configuration item</a> (CI) in Zkybase using Spring Data Neo4j.

Person CI requirements
----------------------

We're going to start really simply here by building a person CI. It's useful to have people in the CMDB for various reasons: they allow you to define fine-grained access controls (e.g., Jim can deploy such-and-such apps to the development environment; Eric can deploy whatever he wants wherever he wants; etc.); they allow you to define groups who will receive notifications for critical events and incidents; etc.

Our person CI will have a username, first and last names, some phone numbers, an e-mail address, a manager, direct reports and finally projects he or she works on. We need to be able to display people in a list view, display a given person in a details view, allow users to create, edit and delete people and so on. Here for example is what the list view will look like, at least for now:

![Person list](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-17-domain-modeling-with-spring-data-neo4j-code/person_list.png)

And here's how our details view will look:

![Person details](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-17-domain-modeling-with-spring-data-neo4j-code/person_details.png)

The relationship between a person and a project has an associated role. This relationship is also the basis for the list of collaborators: two people are collaborators if there's at least one project of which they're both members.

Our simple requirements should be enough to show what it feels like to write Spring Data Neo4j code.

Create the Person and ProjectMembership entities
------------------------------------------------

First we'll create the Person. I've suppressed the validation and JAXB annotations since they're irrelevant for our current purposes:
<pre>package org.skydingo.zkybase.model;

import java.util.Set;
import org.neo4j.graphdb.Direction;
import org.skydingo.zkybase.model.relationship.ProjectMembership;
import org.springframework.data.neo4j.annotation.*;
import org.springframework.data.neo4j.support.index.IndexType;

@NodeEntity
public class Person implements Comparable&lt;Person&gt; {
    @GraphId private Long id;

    @Indexed(indexType = IndexType.FULLTEXT, indexName = "searchByUsername")
    private String username;

    private String firstName, lastName, title, workPhone, mobilePhone, email;

    @RelatedTo(type = "REPORTS_TO")
    private Person manager;

    @RelatedTo(type = "REPORTS_TO", direction = Direction.INCOMING)
    private Set&lt;Person&gt; directReports;

    @RelatedToVia(type = "MEMBER_OF")
    private Set&lt;ProjectMembership&gt; memberships;

    public Long getId() { return id; }

    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }

    public void setUsername(String username) { this.username = username; }

    ... other accessor methods ...

    public Person getManager() { return manager; }

    public void setManager(Person manager) { this.manager = manager; }

    public Set&lt;Person&gt; getDirectReports() { return directReports; }

    public void setDirectReports(Set&lt;Person&gt; directReports) {
        this.directReports = directReports;
    }

    public Iterable&lt;ProjectMembership&gt; getMemberships() { return memberships; }

    public ProjectMembership memberOf(Project project, String role) {
        ProjectMembership membership = new ProjectMembership(this, project, role);
        memberships.add(membership);
        return membership;
    }

    ... equals(), hashCode(), compareTo() ...
}</pre>
There are lots of annotations we're using to put a structure in place. Let's start with nodes and their properties. Then we'll look at simple relationships between nodes. Then we'll look at so-called relationship entities, which are basically fancy relationships. First, here's an abstract representation of our domain model:

![Abstract graph](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-17-domain-modeling-with-spring-data-neo4j-code/abstract_graph.png)

Now let's look at some details.

<strong>Nodes and their properties.</strong> When we have a node-backed entity, first we annotate it with the @NodeEntity annotation. Most of the simple node properties (i.e., properties that aren't relationships to other nodes) come along for the ride. Notice that I didn't have to annotate firstName, lastName, email, and so forth. Spring Data Neo4j will handle the mapping there automatically.

There are a couple of exceptions though. The first one is that I put @GraphId on my id property. This tells Spring Data Neo4j that this is an identifier that we can use for lookups. The other one is the @Indexed annotation, which (surprise) creates an index for the property in question. This is useful when you want an alternative to ID-based lookup.

Now we'll look at relationships. Speaking broadly, there are simple relationships and more advanced relationships. We'll start with the simple ones.

<strong>Simple relationships.</strong> At a low level, Neo4j is a graph database, so we can talk about the graph in graph theoretical terms like nodes, edges, directed edges, DAGs and all that. But here we're using graphs for domain modeling, so we interpret low-level graph concepts in terms of higher-level domain modeling concepts. The language that Spring Data Neo4j uses is "node entity" for nodes, and "relationships" for edges.

Our Person CI has a simple relationship, called REPORTS_TO, that relates people so we can model reporting hierarchies. Person has two fields for this relationship: manager and directReports. These are opposite sites of the same relationship. We use @RelatedTo(type = "REPORTS_TO") to annotate these fields. The annotation has a direction element as well, whose default value is Direction.OUTGOING, which means that "this" node is the edge tail. That's why we specify direction = Direction.INCOMING explicitly for the directReports field.

What's this look like in the database? <a title="Neoclipse site" href="http://wiki.neo4j.org/content/Neoclipse">Neoclipse</a> reveals all. Here are some example reporting relationships (click the image for a larger view):

![Reports-to graph](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-17-domain-modeling-with-spring-data-neo4j-code/reports_to_graph.png)

(Small aside: there's a `@Fetch` annotation--we'll see it in a moment--that tells Spring Data Neo4j to eager load a related entity. For some reason I'm not having to use it for the manager and direct reports relationships, and I'm not sure why. If somebody knows, I'd appreciate the explanation.)

<strong>Relationship entities.</strong> Besides the REPORTS_TO relationship between people, we care about the MEMBER_OF relationship between people and projects. This one's more interesting than the REPORTS_TO relationship because MEMBER_OF has an associated property--role--that's analogous to adding a column to a link table in a RDBMS, as I mentioned in my reply to Brig in the last post. The Person.memberOf() method provides a convenient way to assign a person to a project using a special ProjectMembership "relationship entity". Here's the code:
<pre>package org.skydingo.zkybase.model.relationship;

import org.skydingo.zkybase.model.Person;
import org.skydingo.zkybase.model.Project;
import org.springframework.data.neo4j.annotation.*;

@RelationshipEntity(type = "MEMBER_OF")
public class ProjectMembership {
    @GraphId private Long id;
    @Fetch @StartNode private Person person;
    @Fetch @EndNode private Project project;
    private String role;

    public ProjectMembership() { }

    public ProjectMembership(Person person, Project project, String role) {
        this.person = person;
        this.project = project;
        this.role = role;
    }

    public Person getPerson() { return person; }

    public void setPerson(Person person) { this.person = person; }

    public Project getProject() { return project; }

    public void setProject(Project project) { this.project = project; }

    public String getRole() { return role; }

    public void setRole(String role) { this.role = role; }

    ... equals(), hashCode(), toString() ...

}</pre>
ProjectMembership, like Person, is an entity, but it's a relationship entity. We use @RelationshipEntity(type = "MEMBER_OF") to mark this as a relationship entity, and as with the Person, we use @GraphId for the id property. The @StartNode and @EndNode annotations indicate the edge tail and head, respectively. @Fetch tells Spring Data Neo4j to load the nodes eagerly. By default, Spring Data Neo4j doesn't eagerly load relationships since risks loading the entire graph into memory.

Create the PersonRepository
---------------------------

Here's our PersonRepository interface:

<pre>package org.skydingo.zkybase.repository;

import java.util.Set;
import org.skydingo.zkybase.model.Person;
import org.skydingo.zkybase.model.Project;
import org.springframework.data.neo4j.annotation.Query;
import org.springframework.data.neo4j.repository.GraphRepository;

public interface PersonRepository extends GraphRepository&lt;Person&gt; {

    Person findByUsername(String username);

    @Query("start project=node({0}) match project&lt;--person return person")
    Set&lt;Person&gt; findByProject(Project project);

    @Query(
        "start person=node({0}) " +
        "match person-[:MEMBER_OF]-&gt;project&lt;-[:MEMBER_OF]-collaborator " +
        "return collaborator")
    Set&lt;Person&gt; findCollaborators(Person person);
}</pre>
<p class="size-medium wp-image-215" title="repos">I noted in the last post that all we need to do is extend the GraphRepository interface; Spring Data generates the implementation automatically.</p>

![Spring Data repositories](http://springinpractice.s3.amazonaws.com/blog/images/2011-12-17-domain-modeling-with-spring-data-neo4j-code/repos.png)

For `findByUsername()`, Spring Data can figure out what the intended query is there. For the other two queries, we use `@Query` and the <a title="Cypher query language" href="http://docs.neo4j.org/chunked/milestone/cypher-query-lang.html">Cypher query language</a> to specify the desired result set. The `{0}` in the queries refers to the finder method parameter. In the `findCollaborators()` query, we use `[:MEMBER_OF]` to indicate which relationship we want to follow. These return `Set`s instead of `Iterable`s to eliminate duplicates.

Create the web controller
-------------------------

We won't cover the entire controller here, but we'll cover some representative methods. Assume that we've injected a PersonRepository into the controller.

<strong>Creating a person.</strong> To create a person, we can use the following:
<pre>@RequestMapping(value = "", method = RequestMethod.POST)
public String createPerson(Model model, @ModelAttribute Person person) {
    personRepo.save(person);
    return "redirect:/people?a=created";
}</pre>
Once again, we're ignoring validation. All we have to do is call the save() method on the repository. That's how updates work too.

<strong>Finding all people.</strong> Next, here's how we can get all people:
<pre>@RequestMapping(value = "", method = RequestMethod.GET)
public String getPersonList(Model model) {
    Iterable&lt;Person&gt; personIt = personRepo.findAll();
    List&lt;Person&gt; people =
        new ArrayList&lt;Person&gt;(IteratorUtil.asCollection(personIt));
    Collections.sort(people);
    model.addAttribute(people);
    return "personList";
}</pre>
We have to do some work to get the Iterable that PersonRepository.findAll() returns into the format we want. IteratorUtil, which comes with Neo4j (org.neo4j.helpers.collection.IteratorUtil), helps here.

<strong>Finding a single person.</strong> Here we want to display the personal details we built out above. As with findAll(), we have to do some of the massaging ourselves:
<pre>@RequestMapping(value = "/{username}", method = RequestMethod.GET)
public String getPersonDetails(@PathVariable String username, Model model) {
    Person person = personRepo.findByUsername(username);
    List&lt;ProjectMembership&gt; memberships =
        CollectionsUtil.asList(person.getMemberships());
    List&lt;Person&gt; directReports =
        CollectionsUtil.asList(person.getDirectReports());
    List&lt;Person&gt; collaborators =
        CollectionsUtil.asList(personRepo.findCollaborators(person));

    Collections.sort(directReports);
    Collections.sort(collaborators);

    model.addAttribute(person);
    model.addAttribute("memberships", memberships);
    model.addAttribute("directReports", directReports);
    model.addAttribute("collaborators", collaborators);

    return "personDetails";
}</pre>
If you want to see the JSPs, check out the <a title="Zkybase GitHub site" href="http://skydingo.com/blog/?p=28">Zkybase GitHub site</a>.

Configure the app
-----------------

Finally, here's my beans-service.xml file:
<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:neo4j="http://www.springframework.org/schema/data/neo4j"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://www.springframework.org/schema/data/neo4j
        http://www.springframework.org/schema/data/neo4j/spring-neo4j-2.0.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.0.xsd"&gt;

    &lt;context:property-placeholder
        location="classpath:/spring/environment.properties" /&gt;
    &lt;context:annotation-config /&gt;
    &lt;context:component-scan base-package="org.skydingo.zkybase.service" /&gt;

    &lt;tx:annotation-driven mode="proxy" /&gt;

    &lt;neo4j:config storeDirectory="${graphDb.dir}" /&gt;
    &lt;neo4j:repositories base-package="org.skydingo.zkybase.repository" /&gt;
&lt;/beans&gt;</pre>
Neo4j has a basic POJO-based mapping model and an advanced AspectJ-based mapping model. In this blog post we've been using the basic POJO-based approach, so we don't need to include AspectJ-related configuration like &lt;context:spring-configured /&gt;.

There you have it--a Person CI backed by Neo4j. Happy coding!
