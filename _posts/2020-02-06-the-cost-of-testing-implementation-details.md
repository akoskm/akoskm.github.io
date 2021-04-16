---
title: Writing the right tests can actually save you money
layout: post
type: post
featured: true
date: 2020-02-06 00:00:01
---

Through a platform that hosts coding challenges, I'll show you how to recognize when you're testing implementation details, and tell you why that's not a great idea.

When working with clients, apply these practices to keep development costs optimal, increase confidence in the software, and let the project move faster.

And now for the readers who think that the ultimate cost-saving is to not write tests, ever:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">– We don&#39;t write tests.<br>– Why?<br>– Because we don’t have time for it.<br>– Why?<br>– Because there is too much work and pressure.<br>– Why?<br>– Because we don’t move fast enough.<br>– Why?<br>– Because changing software has become difficult and risky.<br>– Why?<br>– Because we don’t write tests.</p>&mdash; Eduards Sizovs (@eduardsi) <a href="https://twitter.com/eduardsi/status/1381633331230601221?ref_src=twsrc%5Etfw">April 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Now read the whole thing again, with a twist, replace time with money.

Tests coupled to the implementation become useless if the implementation changes. Throwaway Money LLC, basically. After all, organized software development is the science of not burning through your budget as quickly as possible while still making some business impact.

Or, as [@tastapod](https://twitter.com/tastapod) puts it in one of his [talks](https://www.youtube.com/watch?v=4Y0tOi7QWqM):

> ”... sustainably minimizing lead time to business impact and that is the goal of the system of work that we call software development.”

Writing code costs. Whether we add functionality, tests, it costs to write, understand, maintain, improve the code.
Usually, the more code you have the higher the development costs are.

Solving business requirements without creating new code should be ideal, but that's not always an easy task. But how this all relates to testing implementation details and why we should avoid doing it?

Let's get back to our coding challenge platform and try to identify the business requirements in the ”First Factorial” challenge.

The UI is quite intuitively structured, because the panels from left to right separate the requirements, the software, and the tests.

The business requirement here is to write a program that calculates the factorial of an integer.

![Codebite recursion](/assets/posts/images/2020-02-05/recursion.jpg "JavaScript code, calculating the first factorial of an integer with recursion.")

Let's pretend for a moment that we like the idea of testing implementation details. In fact, all we do is test implementation details so this one is going to be easy! We should definitely test the number of recursive function calls for a given input.

Now, fast forward a year.

Let's say we took a course from Data Structures an Algorithms and we learned we can turn recursions into loops.
We come up with this brand new way of calculating factorials:

![Codebite loop](/assets/posts/images/2020-02-05/loop.jpg)

We're excited to try this new algorithm, but our implementation specific test won't pass. The recursions are gone!

You'll run into the same problem while developing or maintaining an kind of software over a period of time. Your software development skills improve during that period, but the problems you solved remain the same.
Today we use LibX to make a network request tomorrow we'll be using LibZ. Unless your task is to explicitly test that LibZ made that request, you should never check that.

Such tests don't increase confidence in our software. They only show the presence of a recursion, the presence of a LibZ call or any other implementation detail that could be temporary.

Focus on the business requirements while writing tests so they keep working even after the implementation changes.
In our case, calculating the factorial of an integer was the requirement and not the presence of recursions.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Write tests. Not too many. Mostly integration.</p>&mdash; Guillermo Rauch (@rauchg) <a href="https://twitter.com/rauchg/status/807626710350839808?ref_src=twsrc%5Etfw">December 10, 2016</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Do you or your team write tests at all? Are you thinking about what part of the software you should test?

Let’s continue the discussion on [Twitter](https://twitter.com/akoskm/status/1225367339836878850) or in the comments below.