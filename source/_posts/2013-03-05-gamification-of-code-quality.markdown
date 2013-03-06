---
layout: post
title: "Gamification of Code Quality"
date: 2013-03-05 20:35
comments: false
categories: ['TDD','development']
---

Quality is one of the most difficult things to define.  There have been [entire books written](http://en.wikipedia.org/wiki/Zen_and_the_Art_of_Motorcycle_Maintenance) about the topic.

Unsurprisingly, it's also difficult to define when it comes to software.  However, most developers will readily agree that writing [unit tests](http://en.wikipedia.org/wiki/Unit_testing) and practicing [test-driven development](http://en.wikipedia.org/wiki/Test-driven_development) are tools for writing higher quality software. 

### Tests are great, why aren't you writing them?

After a few laps on a recent project, I noticed that despite our "test-driven development" goals, our test coverage was lacking.  We were hoving around 50% lines/50% methods complete.

After prodding our developers and "vowing to get better", it wasn't happening.  So I came up with a new plan.

### The Test Coverage Game

We implemented a friendly game on the team to improve test coverage.

1. Invent a scoring system to award "quality points" to developers.

2. Display scores in public locations, but keep it lighthearted.

3. A program collates the results and awards points to the team members after each build.

We focused the scoring and made it simple:

* 1 point for covering a line that was not previously covered by a unit test.
* 3 points for covering a method that was not previously covered by a unit test.
* 5 points for covering a class that was not previously covered by a unit test.

#### Tools

* [Cobertura](http://cobertura.sourceforge.net) to track code coverage.
* [Hudson/Jenkins](http://www.hudson-ci.org) for builds.  
* A custom script to parse the coverage results (coming soon).
* A *whiteboard* for posting scores.

#### Does it work?

After one two-week lap, code coverage jumped from 50% to 60%. It continued to climb as developers strove to contribute and wanted their contributions to be visible to the public.

Universally, it was welcomed by the team, and after a month, test coverage was at 70% -- and developers enjoyed getting there.  It had a successful, nearly bug free launch, too.

### Percentage-based scoring

Lines of code is a bad way to judge a developer's effectiveness: removing lines of code is often a way to improve software quality.  So, a better scoring system would track percentages of the project that are covered by tests rather than lines of code.

Due to the success of the code coverage game, we decided to roll out a percentage-based scoring system to all projects at the company.