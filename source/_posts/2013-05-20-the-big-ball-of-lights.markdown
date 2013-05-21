---
layout: post
title: "The Big Ball of Lights"
date: 2013-05-20 19:08
comments: false
categories: ['development', 'rest', 'graphite']
---

Year after year, your software grows. Developers are building new systems more quickly than old ones are retired.

Eventually, if you don't periodically re-think your core architecture, your mix of services, queues, and applications will probably look like this:

![Christmas Vacation Ball of Lights](/assets/2013-05-20/ball-of-lights.jpg)

Of course, you may not realize how tangled it really is until you decide to upgrade or migrate some key part of the infrastructure.

If you're lucky, you may end up having this conversation *prior to deployment*:

You: *"So, we're moving 'Awesome Web Service' to a new API revision this weekend."*  
Developer: *"Cool. Did you notify Team Shadow?"*  
You: *"No. Why would they care?"*  
Developer: *"A few years ago, they needed some data from the 'Awesome Web Service', so they started using it. And they didn't tell anybody."*

When this occurs, you'll be tempted to pull a Rusty Griswold and give up out of frustration. Next time, you should consider these approaches when you build your next great API.

<!-- more -->

## Implement a Tracking Strategy.

Plan for the scenario that someone is going to use your services *without your knowledge*.  

One strategy is to enforce that every HTTP request includes tracking information in the HTTP headers. (For simple queries from web browsers, you can also allow the caller to pass them as query parameters or form data.)

If you don't want to implement a true authentication/authorization model, you can simply use a scheme like this:

	X-MyCompany-CallingApplicationName: NeatApp
	X-MyCompany-CallingApplicationInstance: NeatApp-Customer-X
	X-MyCompany-CallingUser: bill@customer-x.com

(An API key scheme would be preferable but more complicated; the tradeoff being that you can track users at a much finer grain.)

## Enforce and Emit Tracking Data.

Once you've done this, you can route all requests through a reverse proxy or enterprise service bus to enforce this scheme on every HTTP request.

If the fields are not there, the ESB can respond with a `400 Bad Request` code, thus enforcing your scheme. 

Once enforced, you can use several methods for tracking this information. 

### Basic: Log it.

You can set up your web server of choice to log the custom HTTP headers.  For Tomcat, here's one example access log valve:

	<Valve
    	className="org.apache.catalina.valves.AccessLogValve"
    	directory="${catalina.base}/logs"
    	prefix="access"
    	fileDateFormat="yyyy-MM-dd.HH"
    	suffix=".log"
    	pattern="%h %l %u %t &quot;%r&quote; %s %b NAME:%{X-MyCompany-CallingApplicationName}i INSTANCE:%{X-MyCompany-CallingApplicationInstance}i USER:%{X-MyCompany-CallingUser}i"
	/>


Parse the access.log, and you've got a way to tell who is calling your service.

	
### Better: Emit to StatsD/Graphite.

The central enterprise service bus can emit the data via UDP to a [StatsD/Graphite server](http://codeascraft.com/2011/02/15/measure-anything-measure-everything/) with a naming scheme that parses the URL and includes the calling app name and instance.

For example: `services.{domain}.{callingAppName}.{callingAppInstance}`

This would give you a very quick and easy way to see which applications are calling which services in your infrastructure; also, it would not interfere with throughput or introduce unreliability since it's using a fire-and-forget UDP packet.

A simple metric of `stats.services.awesome-service.\*.\*` yields a dynamic graph that shows all clients of Awesome Service.

![Graphite graph](/assets/2013-05-20/graphite.png)

### Great: Map a Service Registry.

Once you have the data, you can use something like [d3.js](http://d3js.org) to draw a [map of all of your applications](http://www.findtheconversation.com/concept-map).  

(You could plug it into a CMDB, if you have one, as well.)

## Knowledge is power.

And with that bit of foresight, the big ball of lights isn't intimidating. You can make your infrastructure changes with confidence because and you'll know *exactly the impact* of a given change.

