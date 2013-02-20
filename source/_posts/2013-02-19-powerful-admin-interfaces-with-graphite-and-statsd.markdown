---
layout: post
title: "Powerful Admin Interfaces with Graphite and StatsD"
date: 2013-02-19 22:02
comments: false
categories: [graphite, devops]
---

[Graphite](http://graphite.wikidot.com) paired with [StatsD](http://github.com/etsy/statsd) is a simple, powerful way to get dramatic insight into your application's behavior.

The seminal [Etsy blog post](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/) on this subject explains it well:

{% blockquote Ian Malpass, Etsy http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/ Measure Anything, Measure Everything %}  
If Engineering at Etsy has a religion, it’s the Church of Graphs. If it moves, we track it. Sometimes we’ll draw a graph of something that isn’t moving yet, just in case it decides to make a run for it. 
{% endblockquote %}

####With Graphite, you get:
* Beautiful, near real-time graphs of metrics.
* A simply awesome API to embed graphs and source data in nearly any other system.

####Combined with StatsD:
* Fire-and-forget (UDP-based) metric data emission and aggregation.
* Metrics that make sense.
* Brain dead simple API that works in any language, from Groovy to shell scripts.


### Knowledge is Power
What are the first items that get cut from most software projects?  _Effective support and administrative tools._    

Without easy tools at your disposal, how many times have you written some code and thought: 

{% blockquote %}
"What do I do if something goes wrong? Hmm, that's hard. I guess I'll just log the error for now."
{% endblockquote %}
 
I'm certainly just as guilty of this transgression as anyone else.  Why would I waste valuable time writing a screen to manage the failures -- something that may never be used?

<!--more-->

### A Simple Answer
What if the code was this simple:

{% codeblock lang:java %}
private StatsDClient statsdClient;

public boolean doLogin(String user, String pass) {
	boolean result = loginService.doLogin(user, pass);
	if (result) {
		statsdClient.incrementCounter("login.success");  // Nice.
		return true;
	} else {
		log.warn("User " + user + " failed login."); 
		statsdClient.incrementCounter("login.failed");  // Nefarious attempt? Not sure.
		return false;
	}
}
{% endcodeblock %}

Two additional lines of code (if you don't count injecting the `statsdClient`) and all of a sudden you've got a incredibly useful graph like Etsy's: 

![Graph showing login failures](http://etsycodeascraft.files.wordpress.com/2011/02/logins2.png?w=500&h=300)

When it's that easy, _developers will use it by default._  You won't have to include tracking these things in project estimates, steering committees, and testing plans.  It will _just happen_.

### Show off your graphs. Publicly.

Because Graphite has a [fantastic API](https://graphite.readthedocs.org/en/latest/render_api.html), all you need to do in order to show the number of failed logins in your admin interface is include a URL to Graphite's render API in an `<img>` tag in your Admin interface.

{% codeblock lang:html %}
<img src="http://graphite.mycompany.com/render?target=stats.counts.login.success&from=-6hours&format=png">
{% endcodeblock %}

With three lines of code, you've created a real time graph in your admin interface that shows login activity.

Some other places that are great to show a graph:

* A large TV in your workspace.
* The app's Wiki page and documentation.
* In an email to team members.

When a key graph is shared like this, the entire team -- developers, system admins, and support engineers -- will use it.  After a time, they will find it indespensable -- because it will make them more productive.


