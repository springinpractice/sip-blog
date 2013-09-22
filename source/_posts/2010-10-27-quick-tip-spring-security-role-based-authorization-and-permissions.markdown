---
layout: post
title: "Quick tip: Spring Security role-based authorization and permissions"
date: 2010-10-27 12:40:13
comments: true
categories: [Chapter 07 - Authorization, Quick Tips]
---
<h3>The problem: hardcoded role-based authorization</h3>

One of the challenges around using Spring Security is that the examples&mdash;both in the documentation and on the web&mdash;tend to promote an overly-simple approach to role-based authorization, hardcoding roles in the source in a non-configurable fashion. For example:

<pre style="margin:20px 0;">
@PreAuthorize("hasRole('facultyMember')")
public Newsletter getFacultyNews() { ... }
</pre>

(Assume for the sake of example that ACL-based authorization is overkill for the method in question. The user either has permission to read faculty newsletters or not.)

The problem is that when we decide to make a change&mdash;for example, maybe teaching assistants should be allowed to read the faculty newsletters too&mdash;we have to go into the code to make a change:

<pre style="margin:20px 0;">
@PreAuthorize("hasRole('facultyMember') or hasRole('teachingAssistant')")
public Newsletter getFacultyNews() { ... }
</pre>

For domain object security there's no problem because the permissions are cleanly separated from roles. We can map associate individual permissions on domain objects with users and roles as we wish. So the code contains annotations like

<pre style="margin:20px 0;">
@PreAuthorize("hasPermission(#message, write)")
public void editMessage(Message message) { ... }
</pre>

and all is good. We probably won't need to change the relationship between the permission and the method itself; we'll only need to change who (which users/roles) actually has the write permission on the message in question, and we can do that in the database. So that is nice, and we want the same thing for role-based authorization.

<h3>Solution: use granted authorities to model <i>permissions</i>, not roles</h3>

Here we assume an authentication source that models the desired relationship between users, roles and permissions. The typical relationship would be a many-many relationship between users and roles, and a many-many relationship between roles and permissions. For example:

<img src="http://springinpractice.s3.amazonaws.com/springsecurity/refcard106/sample_user_schema.png" alt="Sample custom user schema" />

It would be possible to have a direct relationship between users and permissions too (say to allow for the assignment of fine-grained permissions to specific users in addition to assigning roles), if that were desired.

The schema can be part of some standard authentication source or it can be a custom <code>UserDetailsService</code>; it doesn't matter.

At the end of the day we need to transform our user representation into a <code>UserDetails</code>, and the trick is to map <i>permissions</i>&mdash;not roles&mdash;to <code>GrantedAuthority</code> objects to support the <code>getAuthorities()</code> contract on the <code>UserDetails</code> interface. We still have roles, but they matter only insofar as they help to bundle permissions up into convenient packages. The <code>UserDetails</code> implementation will probably expose the roles, but the <code>UserDetails</code> interface simply exposes the permissions (not the roles) via the <code>getAuthorities()</code> method.

It's really that simple, and the final result is that we can avoid hardcoding roles in the code:

<pre style="margin:20px 0;">
@PreAuthorize("hasRole('PERM_READ_FACULTY_NEWS')")
public Newsletter getFacultyNews() { ... }
</pre>

As an aside, the predicate name <code>hasRole</code> rather than <code>hasAuthority</code> is a minor annoyance since permissions aren't roles. The backing check is against a <code>GrantedAuthority</code> and so <code>hasRole()</code> seems to reflect either the intended or the typical use of <code>GrantedAuthority</code>.
