---
layout: post
title: "The Big Ball of Lights"
date: 2013-05-20 19:08
comments: false
categories: ['development', 'rest', 'graphite']
---

Year after year, your software grows. Developers are building new systems more quickly than old ones are retired.

Eventually, your mix of services, queues, and applications will probably look like this:

![Christmas Vacation Ball of Lights](/assets/2013-05-20/ball-of-lights.jpg)

Of course, you may not realize how tangled it really is until you decide to upgrade or migrate some key part of the infrastructure.

If you're lucky, you may end up having this conversation *prior to deployment*:

{% blockquote %}
You: "We're updating 'Awesome Web Service' to a new API revision this weekend."
Developer: "Cool. Did you notify Team Clueless?"
You: "No. Why would they care?" 
Developer: "Last year, they needed some data from the 'Awesome Web Service', so they started using it. And they didn't tell anybody and didn't update the documentation."
{% endblockquote %}

When this occurs, you'll be tempted, a la Rusty Griswold, to give up out of frustration.  

If you had thought to implement a basic HTTP call tracking strategy, this would not have happened!

<!-- more -->

## Plan for Unexpected Use.

Plan for the scenario that someone is going to use your services *without your knowledge*.  

One strategy is to enforce that every HTTP request includes tracking information in the HTTP headers. (For simple queries from web browsers, you can also allow the caller to pass them as query parameters or form data.)

If you don't want to implement a true authentication/authorization model, you can simply use a scheme like this:

	X-MyCompany-CallingApplicationName: NeatApp
	X-MyCompany-CallingApplicationInstance: NeatApp-Customer-X
	X-MyCompany-CallingUser: bill@customer-x.com

(An API key scheme would be preferable but more complicated; the tradeoff being that you can track users at a much finer grain.)

## Enforce and Emit Tracking Data.

Once you've done this, you can route all requests through a reverse proxy or enterprise service bus to enforce this scheme on every HTTP request.

If the fields are not there, the ESB can respond with a `400 Bad Request` or `403 Forbidden` response, thus enforcing your scheme. 

Once enforced, you can use several methods for tracking this information. 

### Step 1: Log it.

This may seem obvious, but uou can set up your web server of choice to automatically log the custom HTTP headers. In Apache Tomcat, you configure an AccessLogValve like this:

	<Valve
    	className="org.apache.catalina.valves.AccessLogValve"
    	directory="${catalina.base}/logs"
    	prefix="access"
    	fileDateFormat="yyyy-MM-dd.HH"
    	suffix=".log"
    	pattern="%h %l %u %t &quot;%r&quote; %s %b NAME:%{X-MyCompany-CallingApplicationName}i INSTANCE:%{X-MyCompany-CallingApplicationInstance}i USER:%{X-MyCompany-CallingUser}i"
	/>

Parse the access log, and you can find out who is calling Awesome Service.

	
### Step 2: Emit to StatsD/Graphite.

The central enterprise service bus can emit the data via UDP to a [StatsD/Graphite server](http://codeascraft.com/2011/02/15/measure-anything-measure-everything/) with a naming scheme that parses the URL and includes the calling app name and instance.

For example: `services.{domain}.{callingAppName}.{callingAppInstance}`

This would give you a very quick and easy way to see which applications are calling which services in your infrastructure; also, it would not interfere with throughput or introduce unreliability since it's using a fire-and-forget UDP packet.

A simple metric of `stats.services.awesome-service.\*.\*` yields a dynamic graph that shows all clients of Awesome Service.

![Graphite graph](/assets/2013-05-20/graphite.png)

### Step 3: Map a Service Registry.

Once you have the data, you can use something like [d3.js](http://d3js.org) to draw a [map of all of your applications](http://www.findtheconversation.com/concept-map).  

(You could plug it into a CMDB, if you have one, as well.)

## Knowledge is power.

At this point, your graph of application dependencies is updated automatically as the web service is used.

And, armed with this graph, the big ball of lights is just as tangled -- but at least you know where to look to get it unravelled.