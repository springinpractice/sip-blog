---
layout: post
title: "Pagination and sorting with Spring Data JPA"
date: 2012-05-11 01:31
comments: true
categories: [Chapter 02 - Data, Chapter 12 - Articles, Tutorials]
---
In an [earlier post](http://springinpractice.com/2012/04/24/autogenerate-daos-and-queries-using-spring-data-jpa/) I introduced Spring Data JPA, which makes it really easy to create a DAO layer. I didn't get into too much depth, so this time I want to explore a couple of cool features that the DAOs support: pagination and sorting.

<!-- more -->

Pagination and sorting are useful when you have long lists that you want the user to be able to navigate. Here for example is a UI for a runbook app I'm building. One of the things it allows the user to do is view deployment logs, which we typically want to see in reverse chronological order. Also, since there are lots of logs, we want to page.

![Pagination screenshot](http://springinpractice.s3.amazonaws.com/blog/images/2012-05-11-pagination-and-sorting-with-spring-data-jpa/pagination-1.jpg)

There are different ways to design a pagination system from a user experience perspective. Here I've done something pretty typical: I have links for first/previous/next/last, and then I show a bounded set of pages around the current page.

The repository
--------------

How does Spring Data JPA help? Here's my `DeploymentRepo` interface:

    package com.example.repo;
    
    import org.springframework.data.jpa.repository.JpaRepository;
    import com.example.model.Deployment;
    
    public interface DeploymentRepo extends JpaRepository<Deployment, Long> { }

The `JpaRepository` interface extends Spring Data's `PagingAndSortingRepository` interface, so I get some paging/sorting finders for free.

The service
-----------

I have a simple service bean that calls the repo:

    package com.example.service.impl;
    
    import javax.inject.Inject;
    
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;
    import org.springframework.data.domain.Sort;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    
    import com.example.repo.DeploymentRepo;
    import com.example.model.Deployment;
    import com.example.service.DeploymentLogService;
    
    @Service
    @Transactional
    public class DeploymentLogServiceImpl implements DeploymentLogService {
        private static final int PAGE_SIZE = 50;
        
        @Inject private DeploymentRepo deploymentRepo;
    
        public Page<Deployment> getDeploymentLog(Integer pageNumber) {
            PageRequest request =
                new PageRequest(pageNumber - 1, PAGE_SIZE, Sort.Direction.DESC, "startTime");
            return deploymentRepo.findAll(pageRequest);
        }
    }

Spring Data uses 0-indexed pages, but I want my service interface to use 1-indexed pages (they will be user-visible and I want the page numbers to be intuitive), so I make the appropriate adjustment in the request. I specify the page size (50 deployments per page), sort direction, and also one or more property names to act as sort keys. Here I've chosen `startTime`, which is a timestamp for the start of the deployment.

That takes care of the Spring Data JPA part, but just for fun, I'll show you a simplified version of the controller method and JSP too.

The controller
--------------

Here's the controller method:

    @RequestMapping(value = "/pages/{pageNumber}", method = RequestMethod.GET)
    public String getRunbookPage(@PathVariable Integer pageNumber, Model model) {
        Page<Deployment> page = deploymentService.getDeploymentLog(pageNumber);
        
        int current = page.getNumber() + 1;
        int begin = Math.max(1, current - 5);
        int end = Math.min(begin + 10, page.getTotalPages());
        
        model.addAttribute("deploymentLog", page);
        model.addAttribute("beginIndex", begin);
        model.addAttribute("endIndex", end);
        model.addAttribute("currentIndex", current);
        
        return "deploymentLog";
    }

Note again that I've adjusted the page numbers to convert Spring Data's 0-indexing to my app's 1-indexing.

I've precalculated the begin/end indices because JSTL doesn't have the min and max functions, and anyway, it's cleaner to do this sort of thing in the controller.

The JSP
-------

Finally here's the page navigation in the JSP. It uses the [Twitter Bootstrap](http://twitter.github.com/bootstrap/) library for the UI, so that's where the various CSS elements come from.

    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    
    <c:url var="firstUrl" value="/pages/1" />
    <c:url var="lastUrl" value="/pages/${deploymentLog.totalPages}" />
    <c:url var="prevUrl" value="/pages/${currentIndex - 1}" />
    <c:url var="nextUrl" value="/pages/${currentIndex + 1}" />
    
    <div class="pagination">
        <ul>
            <c:choose>
                <c:when test="${currentIndex == 1}">
                    <li class="disabled"><a href="#">&lt;&lt;</a></li>
                    <li class="disabled"><a href="#">&lt;</a></li>
                </c:when>
                <c:otherwise>
                    <li><a href="${firstUrl}">&lt;&lt;</a></li>
                    <li><a href="${prevUrl}">&lt;</a></li>
                </c:otherwise>
            </c:choose>
            <c:forEach var="i" begin="${beginIndex}" end="${endIndex}">
                <c:url var="pageUrl" value="/pages/${i}" />
                <c:choose>
                    <c:when test="${i == currentIndex}">
                        <li class="active"><a href="${pageUrl}"><c:out value="${i}" /></a></li>
                    </c:when>
                    <c:otherwise>
                        <li><a href="${pageUrl}"><c:out value="${i}" /></a></li>
                    </c:otherwise>
                </c:choose>
            </c:forEach>
            <c:choose>
                <c:when test="${currentIndex == deploymentLog.totalPages}">
                    <li class="disabled"><a href="#">&gt;</a></li>
                    <li class="disabled"><a href="#">&gt;&gt;</a></li>
                </c:when>
                <c:otherwise>
                    <li><a href="${nextUrl}">&gt;</a></li>
                    <li><a href="${lastUrl}">&gt;&gt;</a></li>
                </c:otherwise>
            </c:choose>
        </ul>
    </div>

Spring Data JPA makes it very nice and simple. And Twitter Bootstrap looks great too.
