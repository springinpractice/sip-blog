---
layout: post
title: "Optimistic locking with Spring Data REST"
date: 2013-09-14 20:43
comments: true
categories: [Chapter 02 - Data, Chapter 13 - Integration]
---
I'm working on a web service for a document management system, where clients grab documents from the web service, modify them and submit the updates. Since multiple clients can all grab the same document at the same time, we needed to implement an [optimistic locking](http://c2.com/cgi/wiki?OptimisticLocking) scheme. In this scheme, each document has a version number, and when the client submits an update to the service, the service checks to see whether the submitted version number baseline is still the most recent one in the database. If so, we increment the version number and the update proceeds. Otherwise, we throw an exception indicating a conflict.

<!-- more -->

From a technology perspective we're using [Spring Data REST](http://projects.spring.io/spring-data-rest/), [Spring Data JPA](http://projects.spring.io/spring-data-jpa/), JPA and Hibernate to implement the web service. So I originally tried to use [JPA's @Version annotation](http://docs.oracle.com/javaee/6/api/javax/persistence/Version.html). But for reasons I describe [here](http://stackoverflow.com/questions/18780621/does-spring-data-rest-support-jpa-version) it doesn't work. [Marten Deinum](https://twitter.com/mdeinum) notes further that this isn't really a problem with Spring Data REST *per se*; it affects e.g. form submissions too. At Marten's suggestion, I created an [enhancement request](https://jira.springsource.org/browse/DATAREST-160), but since I needed something now, I set out to implement optimisitic locking for Spring Data REST without `@Version`.

The trick is to listen for events that are fired before the update occurs. Spring Data REST fires such events, but so does JPA, and I decided to use JPA's events instead to ensure that the version number get/test/increment happens as part of a single atomic transaction. (My guess is that the Spring Data REST [BeforeSaveEvent](http://docs.spring.io/spring-data/rest/docs/1.1.0.M1/reference/htmlsingle/#events-chapter) fires before entering the transaction.)

Here's the code. First, here's an interface for versioned entities.

    package myapp.entity;
    
    public interface VersionedEntity {

        Long getVersion();

        void setVersion(Long version);
    }

Next we have an abstract base class for versioned entities. I could have done a mixin-style implementation here (using, e.g., [AspectJ inter-type declarations](http://www.eclipse.org/aspectj/doc/next/progguide/language-interType.html)), but I decided to keep it simple for now.

    package myapp.entity;
    
    import javax.persistence.Column;
    import javax.persistence.EntityListeners;
    import javax.persistence.MappedSuperclass;
    import myapp.repo.listener.OptimisticLockListener;
    
    @MappedSuperclass
    @EntityListeners(OptimisticLockListener.class)
    public abstract class AbstractVersionedEntity implements VersionedEntity {
    	   
        @Column(name = "VERSION")
        private Long version;
       
        @Override
        public Long getVersion() { return version; }
       
        @Override
        public void setVersion(Long version) { this.version = version; }
    }

Notice the `@EntityListener` annotation. This tells JPA which class will listen for JPA lifecycle events. Here's the `OptimisticLockListener`.

    package myapp.repo.listener;
    
    import javax.persistence.PreUpdate;
    import org.springframework.stereotype.Component;
    import myapp.entity.VersionedEntity;
    import myapp.util.ApplicationContextProvider;
    
    public class OptimisticLockListener {
        
        @PreUpdate
        public void preUpdate(Object entity) {
            if (entity instanceof VersionedEntity) {
                getChecker().check((VersionedEntity) entity);
            }
        }
        
        private OptimisticLockChecker getChecker() {
            return ApplicationContextProvider
                .getApplicationContext()
                .getBean(OptimisticLockChecker.class);
        }
    }

In the listing above we grab an `OptimisticLockChecker` and then run the check. I've implemented that as a separate class because I'm going to need the application's JPA `EntityManager` to do the version check, and I need a managed bean to use `@PersistenceContext` to inject the `EntityManager`. Unfortunately, [JPA 2.0 doesn't treat entity listeners as managed beans](http://stackoverflow.com/questions/12951701/how-to-get-entity-manager-or-transaction-in-jpa-listener) (apparently that will change in JPA 2.1). That's why there's a separate `OptimisticLockChecker` class.

[Venkatt Guhesan](https://twitter.com/vguhesan) offers `ApplicationContextProvider` as a [clever way to get the app's context from an unmanaged instance](http://mythinkpond.wordpress.com/2010/03/22/spring-application-context/). (Note that we can't simply create a new `ClassPathXmlApplicationContext` from the Spring configuration files since we want to use the same `EntityManagerFactory` that the rest of the app is using.) Here's the technique.

    package myapp.util;
    
    import org.springframework.beans.BeansException;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    
    public class ApplicationContextProvider implements ApplicationContextAware {
        private static ApplicationContext applicationContext;
        
        public static ApplicationContext getApplicationContext() { return applicationContext; }
        
        @Override
        public void setApplicationContext(ApplicationContext appContext) throws BeansException {
            applicationContext = appContext;
        }
    }

Finally, let's look at the `OptimisticLockChecker` itself.

    package myapp.repo.listener;
    
    import javax.persistence.EntityManager;
    import javax.persistence.OptimisticLockException;
    import javax.persistence.PersistenceContext;
    import org.springframework.beans.BeanUtils;
    import org.springframework.stereotype.Component;
    import myapp.entity.VersionedEntity;
    
    @Component
    public class OptimisticLockChecker {
        @PersistenceContext private EntityManager entityManager;
        
        public void check(VersionedEntity entity) {
            Long submittedVersion = entity.getVersion();
            if (submittedVersion == null) {
                throw new RuntimeException("Submitted entity must have a version");
            }
    
            Object copy = BeanUtils.instantiate(entity.getClass());
            BeanUtils.copyProperties(entity, copy);
            entityManager.refresh(entity);
            Long latestVersion = entity.getVersion();
            BeanUtils.copyProperties(copy, entity);

            if (submittedVersion != latestVersion) {
                throw new OptimisticLockException(
                        "Stale entity: submitted version " + submittedVersion
                        + ", but latest version is " + latestVersion);
            }
            
            entity.setVersion(entity.getVersion() + 1);
        }
    }

Despite appearances, the code is a little tricky because we're trying to compare an entity version in the persistence context with an entity version in the database, and by design JPA hides the database from the developer. While we can certainly use JDBC to grab the entity version in the database, I wanted to avoid having to map all the entities to "get-latest-version" SQL queries since we already have all the table names mapped in the entity classes. I suppose I could have read the contents of the `@Table` annotation and created dynamic queries, but I wanted to do something cleaner.

When we get the entity, we cache a copy using Spring's `BeanUtils.copyProperties()` utility. That allows us to refresh the original entity, which puts it back in its database state, allowing us to see the latest version number in the database. After that, we put the entity back into its modified state by doing another `BeanUtils.copyProperties()`, this time from the copy to the entity.

(Instead of cloning, I would have preferred to detach the entity, load the database entity into the persistence context, compare the two, and then reattach the entity to the persistence context. Unfortunately, while Hibernate supports reattachment, [JPA doesn't](http://stackoverflow.com/questions/912659/what-is-the-proper-way-to-re-attach-detached-objects-in-hibernate). JPA supports merges, but this leaves the original entity detached, which is no good since that's the reference that JPA is going to try to save after the entity listener's `@PreUpdate` handler completes its processing. See [this article](http://java.dzone.com/articles/saving_detatched_entities) for more information on these topics.)

Now our entity is back to its original state, and we have both the submitted and the latest version numbers. So we compare and either throw an exception or else increment the version number.

Perhaps the Spring Data REST guys will provide more direct support for `@Version` (or some suitable alternative) at some future point. Until then, I hope the approach above proves useful.
