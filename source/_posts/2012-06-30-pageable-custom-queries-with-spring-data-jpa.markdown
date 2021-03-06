---
layout: post
title: "Pageable custom queries with Spring Data JPA"
date: 2012-06-30 12:41
comments: true
categories: [Chapter 02 - Data, Quick Tips]
---
In previous posts I explained how you can use Spring Data JPA to [create repositories that support custom queries](http://springinpractice.com/2012/04/24/autogenerate-daos-and-queries-using-spring-data-jpa/), as well as to [support paging in your app](http://springinpractice.com/2012/05/11/pagination-and-sorting-with-spring-data-jpa/). You might wonder whether you can use these together.

The answer is yes. It works just like you would expect:

    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.jpa.repository.JpaRepository;
    
    public interface IncidentRepo extends JpaRepository<Incident, Long> {
    
        Page<Incident> findByProblemId(Long problemId, Pageable pageable);
    }
