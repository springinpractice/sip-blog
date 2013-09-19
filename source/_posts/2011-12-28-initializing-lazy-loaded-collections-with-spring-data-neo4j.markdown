---
link: http://springinpractice.com/2011/12/28/initializing-lazy-loaded-collections-with-spring-data-neo4j/
layout: post
title: Initializing lazy-loaded collections with Spring Data Neo4j
date: 2011-12-28 01:22:03
comments: true
categories: [Chapter 02 - Data, Chapter 11 - CMDB, Quick Tips]
---
Spring Data Neo4j has been around for about a year now, but the so-called "simple mapping" strategy (the one that doesn't use AspectJ) is new. So it doesn't have some of the bells and whistles that you might expect to see if you're coming from, say, Hibernate. In particular, it doesn't (as of Spring Data Neo4j 2.0.0.RELEASE) use proxies for collections.

The <a href="http://stackoverflow.com/questions/8218864/fetch-annotation-in-sdg-2-0-fetching-strategy-questions">question of how to initialize lazily loaded collections</a> came up on StackOverflow. Michael Hunger (Spring Data Neo4j lead) explained that you can use the Neo4jTemplate's fetch() method to perform the initialization. Here's some code showing how to do it:
<pre>package org.skydingo.skybase.service.impl;

import javax.inject.Inject;
import org.skydingo.skybase.model.Person;
import org.skydingo.skybase.repository.PersonRepository;
import org.skydingo.skybase.service.PersonService;
import org.springframework.data.neo4j.support.Neo4jTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class PersonServiceImpl implements PersonService {
    @Inject private Neo4jTemplate template;
    @Inject private PersonRepository personRepo;

    @Override
    public Person findPersonDetails(Long id) {
        Person person = personRepo.findOne(id);
        template.fetch(person.getDirectReports());
        return person;
    }

    ... other methods ...
}</pre>

Have fun.
