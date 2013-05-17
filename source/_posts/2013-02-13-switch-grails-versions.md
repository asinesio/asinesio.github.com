---
layout: post
title: "Switching Grails Versions"
tagline: "Another annoyance out of the way."
description: "A small script to help Grails developers switch versions."
categories: [grails, script]
---

This is my #firstpost, so I thought I'd share a useful Grails shell script.

(**Update: you really should use [gvm](http://gvmtool.net) instead of this script.** It is a similar idea but also handles installation, path setup, and other tools like Gradle, vert.x, and Griffon.)

One of the annoying things about using [Grails](http://www.grails.org) from the command line is dealing with multiple Grails versions on your system.

This isn't an issue if you're using an IDE that can refer to a Grails version outside of your $PATH.  However, I find that using a command-line Grails in combination with an IDE like STS or GGTS is the most productive way for me to write Grails code.


<!--more-->

If you do nothing special, you'll find yourself adding the following to your `~/.bash_profile`  to make Grails work from a command line:

	GRAILS_HOME = /Users/asinesio/dev/tools/grails-2.1.0
	export PATH=$PATH:$GRAILS_HOME/bin

Of course, you now have to change your `.bash_profile` every time you switch projects. Fail.

The obvious solution is a simple shell script that leverages the following ideas:

1. Use a symbolic link from something like `/Users/asinesio/dev/tools/grails` to the actual Grails version you want to use.
2. Set this link as `GRAILS_HOME`.
3. Add `GRAILS_HOME/bin` to your `PATH`.
4. Use a [shell script](https://gist.github.com/asinesio/4946855) on your path to automatically change the symbolic link to point to the correct Grails version.

Once you've done that, changing is as easy as running: `grails-switch-version.sh 2.2.0`

{% gist 4946855 grails-switch-version.sh %}

