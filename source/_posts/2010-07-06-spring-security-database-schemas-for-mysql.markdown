---
link: http://springinpractice.com/2010/07/06/spring-security-database-schemas-for-mysql/
layout: post
title: Spring Security 3 database schemas for MySQL
date: 2010-07-06 00:36:28
comments: true
categories: [Chapter 06 - Authentication, Chapter 07 - Authorization, Reference]
---
In preparation for a <a href="http://refcardz.dzone.com/refcardz/expression-based-authorization?oid=hom25493">DZone Refcard on Spring Security 3</a> I'm doing, here are the Spring Security database schemas for MySQL.

This isn't intended to be an exhaustive treatment. Rather it's more a reference for the MySQL dialect version of the database schemas, though I included a few brief notes just to help people get a high-level understanding of the schemas. For more detailed information, please consult the <a href="http://static.springsource.org/spring-security/site/reference.html">Spring Security 3 Reference Documentation</a>.

<h3>User schema</h3>
<img src="http://springinpractice.s3.amazonaws.com/springsecurity/dbschemas/users_erd.png" alt="User schema ERD" />

Each user can have zero or more "authorities", or roles. I think Spring Security expects there to be at least one role, though, because login involving roleless users fail.

<pre>create table users (
    username varchar(50) not null primary key,
    password varchar(50) not null,
    enabled boolean not null
) engine = InnoDb;

create table authorities (
    username varchar(50) not null,
    authority varchar(50) not null,
    foreign key (username) references users (username),
    unique index authorities_idx_1 (username, authority)
) engine = InnoDb;
</pre>

<h3 style="margin-top:20px;">Group schema</h3>

The group schema provides a way to organize users into groups and then assign roles to those groups. Each user inherits the roles from his associated groups.

<img src="http://springinpractice.s3.amazonaws.com/springsecurity/dbschemas/groups_erd.png" alt="Group schema ERD" />
<pre>create table groups (
    id bigint unsigned not null auto_increment primary key,
    group_name varchar(50) not null
) engine = InnoDb;

create table group_authorities (
    group_id bigint unsigned not null,
    authority varchar(50) not null,
    foreign key (group_id) references groups (id)
) engine = InnoDb;

create table group_members (
    id bigint unsigned not null auto_increment primary key,
    username varchar(50) not null,
    group_id bigint unsigned not null,
    foreign key (group_id) references groups (id)
) engine = InnoDb;

</pre>

<h3 style="margin-top:20px;">Remember-me schema</h3>

Haven't actually used this one, but from what I understand, this schema supports a hardened, persistent remember-me authentication scheme. Spring Security's default remember-me scheme doesn't use the database at all (just a cookie).

<img src="http://springinpractice.s3.amazonaws.com/springsecurity/dbschemas/remember_erd.png" alt="Remember-me schema ERD" />
<pre>create table persistent_logins (
    username varchar(64) not null,
    series varchar(64) primary key,
    token varchar(64) not null,
    last_used timestamp not null
) engine = InnoDb;

</pre>

<h3 style="margin-top:20px;">ACL schema</h3>

Access control lists (ACLs) might be the most challenging aspect of Spring Security, but the underlying schema is actually pretty straightforward. The concept is that each secure domain object has an associated ACL that defines who can do what with that domain object. Exactly the same idea as permissions on an OS folder, for example.

The SID ("security identity") represents the "who", and comes in both principal and role flavors; that is, the SID can be either an individual user or else it can be a role. The class and object identity, taken together, represent the domain object. Finally, the <code>acl_entry</code> table contains all the ACLs; each access control entry (ACE) specifies an ordered triple &lt;domain object, SID, permission&gt; and indicates whether the permission in question is granted or denied for the SID and domain object. To take an example, SID Ben might have the write permission granted for a particular forum message.

<img src="http://springinpractice.s3.amazonaws.com/springsecurity/dbschemas/acl_erd.png" alt="ACL schema ERD" />
<pre>create table acl_sid (
    id bigint unsigned not null auto_increment primary key,
    principal tinyint(1) not null,
    sid varchar(100) not null,
    unique index acl_sid_idx_1 (sid, principal)
) engine = InnoDb;

create table acl_class (
    id bigint unsigned not null auto_increment primary key,
    class varchar(100) unique not null
) engine = InnoDb;

create table acl_object_identity (
    id bigint unsigned not null auto_increment primary key,
    object_id_class bigint unsigned not null,
    object_id_identity bigint unsigned not null,
    parent_object bigint unsigned,
    owner_sid bigint unsigned,
    entries_inheriting tinyint(1) not null,
    unique index acl_object_identity_idx_1
        (object_id_class, object_id_identity),
    foreign key (object_id_class) references acl_class (id),
    foreign key (parent_object) references acl_object_identity (id),
    foreign key (owner_sid) references acl_sid (id)
) engine = InnoDb;

create table acl_entry (
    id bigint unsigned not null auto_increment primary key,
    acl_object_identity bigint unsigned not null,
    ace_order int unsigned not null,
    sid bigint unsigned not null,
    mask int not null,
    granting tinyint(1) not null,
    audit_success tinyint(1) not null,
    audit_failure tinyint(1) not null,
    unique index acl_entry_idx_1 (acl_object_identity, ace_order),
    foreign key (acl_object_identity)
        references acl_object_identity (id),
    foreign key (sid) references acl_sid (id)
) engine = InnoDb;</pre>

<h3>Sample code</h3>

<ul>
<li><a href="https://github.com/springinpractice/sip06">Spring in Practice, chapter 6 [GitHub]</a> - authentication</li>
<li><a href="https://github.com/springinpractice/sip07">Spring in Practice, chapter 7 [GitHub]</a> - authorization, including ACLs</li>
</ul>
