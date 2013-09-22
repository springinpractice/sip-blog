---
layout: post
title: "Migrating non-Wordpress blog comments into Wordpress"
date: 2012-05-03 12:24:51
comments: true
categories: [Quick Tips]
---
This isn't a Spring post, but I'm doing something that I think others might find useful, so I'm going to share it.

I'm in the process of migrating content over from my old Wheeler Software blog to this one, which is a Wordpress blog. Besides the posts themselves, I want to move the comments over.

The slight wrinkle in the plan is that the software and database for the old blog are custom. So getting the comments over involves some SQL scripting. Both databases are MySQL databases, so that helps a bit.

Here's what I'm doing.

<!-- more -->

<h3>The old, custom comment table</h3>

<pre>CREATE TABLE `services_comment` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `url` varchar(255) NOT NULL DEFAULT '',
  `name` varchar(50) NOT NULL,
  `email` varchar(50) NOT NULL,
  `web` varchar(100) DEFAULT NULL,
  `text` text NOT NULL,
  `html_text` text,
  `ip_addr` varchar(15) NOT NULL,
  `date_created` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `date_modified` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `services_comment_idx0` (`url`,`date_created`)
) ENGINE=InnoDB AUTO_INCREMENT=1686 DEFAULT CHARSET=latin1;</pre>

<h3>The Wordpress post table</h3>

This isn't the entire table, but just the columns that we care about for this post:

<pre>CREATE TABLE `wp_posts` (
  `ID` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `post_title` text NOT NULL,
  `comment_count` bigint(20) NOT NULL DEFAULT '0',

  ... other columns that we don't care about for our current purpose ...

) ENGINE=MyISAM AUTO_INCREMENT=1126 DEFAULT CHARSET=utf8;</pre>

<h3>The Wordpress comment table</h3>

<pre>CREATE TABLE `wp_comments` (
  `comment_ID` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `comment_post_ID` bigint(20) unsigned NOT NULL DEFAULT '0',
  `comment_author` tinytext NOT NULL,
  `comment_author_email` varchar(100) NOT NULL DEFAULT '',
  `comment_author_url` varchar(200) NOT NULL DEFAULT '',
  `comment_author_IP` varchar(100) NOT NULL DEFAULT '',
  `comment_date` datetime NOT NULL DEFAULT '0000-00-00 00:00:00',
  `comment_date_gmt` datetime NOT NULL DEFAULT '0000-00-00 00:00:00',
  `comment_content` text NOT NULL,
  `comment_karma` int(11) NOT NULL DEFAULT '0',
  `comment_approved` varchar(20) NOT NULL DEFAULT '1',
  `comment_agent` varchar(255) NOT NULL DEFAULT '',
  `comment_type` varchar(20) NOT NULL DEFAULT '',
  `comment_parent` bigint(20) unsigned NOT NULL DEFAULT '0',
  `user_id` bigint(20) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`comment_ID`),
  KEY `comment_approved` (`comment_approved`),
  KEY `comment_post_ID` (`comment_post_ID`),
  KEY `comment_approved_date_gmt` (`comment_approved`,`comment_date_gmt`),
  KEY `comment_date_gmt` (`comment_date_gmt`),
  KEY `comment_parent` (`comment_parent`)
) ENGINE=MyISAM AUTO_INCREMENT=289 DEFAULT CHARSET=utf8;</pre>

<h3>The copy script</h3>

<div class="alert warning"><strong>WARNING and DISCLAIMER:</strong> Before you try anything like this, back up your database! And it's a good idea to experiment with your script on a separate copy of the database before trying it out on the real thing.

You've been warned. I'm not responsible if you hose your database, and I'm not going to field support requests!</div>

<h4>Step 1. Copy the old table into the new database.</h4>

I created a copy of my old comment table inside the new Wordpress database to facilitate the copying.

<h4>Step 2. Look up the target Wordpress post's ID</h4>

You'll need to have something to attach your comments to. You can use whatever means you like here; I just looked it up by the post title:

<pre>select
  id
from
  wp_posts
where
  post_title = 'Getting started with Hibernate Validator' and
  post_parent = 0
into
  @post_id;</pre>

Note that you want the post with <code>post_parent = 0</code>. Other posts are the various revisions you create over time, and Wordpress has to have a way to attach all the comments to the same single post.

<h4>Step 3. Copy the comments</h4>

This is obviously dependent on the specifics of your source table. In my case the source table is pretty close to the destination table, so that makes things a lot easier.

I didn't really care about getting the date vs. GMT date right, so I just used the same date for both. If you care, I'm sure there's a function that can handle that.

<pre>insert into
  wp_comments (comment_post_ID, comment_author, comment_author_email, comment_author_url, comment_author_IP, comment_date, comment_date_gmt, comment_content, comment_approved)
select
  @post_id, name, email, web, ip_addr, date_created, date_created, html_text, 1
from
  services_comment
where
  url = '/hibernate-validator.html'
order by
  date_created;</pre>

<h4>Step 4. Update the post's comment count</h4>

Presumably for performance reasons, Wordpress stores the comment count with the post itself. So we have to update that column or else it will say "0 comments" even though there's a bunch of comments below:

<pre>select count(*) from wp_comments where comment_post_ID = @post_id and comment_approved = 1 into @comment_count;
update wp_posts set comment_count = @comment_count where ID = @post_id;</pre>

It probably wouldn't be a bad idea to wrap these up in a stored procedure if you have a lot of posts. But no biggie either way.

<h4>Step 5. Verify</h4>

Check to see whether your comments showed up. Mine did:

<ul class="square">
<li><a href="http://springinpractice.com/2009/02/02/getting-started-with-hibernate-validator/" target="_blank">Hibernate Validator post</a></li>
<li><a href="http://springinpractice.com/2008/05/05/build-a-shopping-cart-with-spring-web-flow-2-part-1/" target="_blank">Spring Web Flow post</a></li>
</ul>

For some reason the Gravatars don't seem to be showing up as much when there are a lot of comments all on a single page. I don't know if it's rate limited or what.
