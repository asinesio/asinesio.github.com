---
layout: post
title: "The Perils of the Grails Searchable Plugin"
date: 2013-02-23 00:05
comments: false
categories: [grails]
---

The [Grails Searchable Plugin](http://grails.org/plugin/searchable) is generally where developers end up when they are looking to add full-text searching to their Grails application.

It's got a lot going for it:

* Simple, embedded Lucene index with minimal initial setup.
* Automatic GORM mirroring of changes.
* Widely used; according to Grails.org, it's installed in almost _7% of Grails projects_.
* Fast and simple, it works out of the box.

Before I go further: know that I really like the Searchable plugin.  It's a great way to get search in a smaller application.
<!--more-->

### Adding Searchable to a Project: A dramatization
You're working on a project.  The end of the lap is near, and your task is to implement a simple search screen.  

You find Searchable and decide to try it out.

Installation's easy. `grails install-plugin searchable` or `:searchable:0.6.4` in your BuildConfig.  Tweak some settings, and within an hour, you've got an embedded full text search engine.  

"Man, that's cool," you think to yourself.  "And so easy to use. I love it."

You proudly show it off to your project sponsors: "Look! Embedded Google, right in the app!  Look fast it is!"

Everyone loves it. Your manager thinks you're a rock star.  Life is great.

Your QA team notices some searches randomly failing while testing the app, but you can't reproduce it.  You research it for hours, can't determine the cause, and eventually blame some random piece of infrastructure for your concerns.

"I know the network guys were scheduled to do that switch firmware upgrade, that must have been it."

#### Then, Production Deployment.

Your users are loving the full text search. Your app is a success and everyone pats you on the back.

Meanwhile, your search index is adding rows by the minute as your database grows.  The random failure from QA is lurking there, in the shadows, planning the perfect moment to jump up and bite you.

#### Six months later... 

...your app suddenly crashes.  No data updates can happen -- database transactions are timing out like crazy.  You frantically turn up logging and throw your full effort at debugging.  Since it's a transaction timeout, you focus on the database.

You've got your ops team, your DBAs, and your network team frantically checking the database to see what could be causing the issue.

After hours of fruitless troubleshooting, you finally remember the bug that you ignored.  Now you see that the "random search problem" wasn't random at all.

Further research reveals that _storing the Compass index on an NFS mount_ was the cause of the problem. It's in the documentation.  While researching, you find:

##### _[Compass](http://www.compass-project.org), a core library used by Searchable, is no longer maintained._

You realize with surprise that the most popular search plugin for Grails uses an obsolete library to map GORM objects to it's Lucene search index.

The author has abandoned Compass and created [Elastic Search](http://www.elasticsearch.org), which, now that you look at it, looks to be a much better solution.
  
##### _Nobody uses Searchable with indexes that are shared and mirrored across multiple app instances._

You weren't a total idiot when you built your app -- you decided to launch several Tomcat instances and load balance them.  In order for updates made in one node to show up in the others, you enabled mirroring in all nodes.

What you didn't realize is that in order to share the index files, you had to use something like an NFS mount to share them among instances.

Then, suddenly, it hits you like a ton of bricks:

##### _Tbe most popular Grails plugin for searching -- Searchable -- is great for a quick solution.  But it is not designed for scaling to multiple instances very well._

You immediately start work porting the app to a different search engine technology -- for example, Apache Solr or Elastic Search -- and cursing the name of Searchable.

### The Moral of the Story

 1. Much like high school kids, the popular plugins are not necessarily the best plugins. 

 2. _Pay attention to what libraries are used by your dependencies before you choose them_ or you may end up with dead-end software in your application.
 

 
