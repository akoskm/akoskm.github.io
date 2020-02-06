---
title: The cost of testing implementation details
layout: post
type: post
date: 2020-02-06 00:00:01
---

Have you ever wondered, what's the goal of software development?

Could we define it as a discipline that solves business requirements within the constraints of budget, schedule, and technology? Maybe.

As [@tastapod](https://twitter.com/tastapod) puts it in one of his [talks](https://www.youtube.com/watch?v=4Y0tOi7QWqM):

> ”... sustainably minimizing lead time to business impact and that is the goal of the system of work that we call software development.”

So, I'm pretty sure it has to do something with goals and accomplishing them over time within some financial or some other constraints.

Writing code costs. Whether we add functionality or tests, it costs to write, understand, maintain, improve, or even introduce it to developers that just joined our team. So more code probably equals higher costs.

Therefore, solving business requirements without creating new code should be ideal, but unfortunately, that's not always an easy task.

So, how this all relates to testing implementation details and why we should avoid doing it?

Through a platform that hosts coding challenges, I'll try to explain why I think you shouldn't focus on testing implementation details.
While such tests increase costs, in the end, they not necessarily tell us if our software still works as it should be. They're tightly coupled to the implementation and not reusable, meaning, if you delete the implementation, you must delete those tests as well.

Have you ever tried programming challenges? Back in the days, Project Euler was popular. All it did, was asking you for a single input.

![Project Euler interface](/assets/posts/images/2020-02-05/euler.jpg)

Looking at the newer platforms (Project Euler started in 2001), not much has changed. They're still telling if you correctly solved their problem, and do nothing more.

Let's take a look at the ”First Factorial” coding challenge.

We can identify some business requirements here. We need to write a program that helps the user to calculate the factorial of an integer.

The panels, from left to right, clearly separate the business requirement, the software, and the tests.

![Codebite recursion](/assets/posts/images/2020-02-05/recursion.jpg)

Above, we have a working software returning the factorial of a positive integer. One way to test this software is to check the number of recursive function calls, but I don't think that's happening under the hood.

That's probably because the problem has more than one solution (which is the case for most of the stuff we do in real-world applications). Even if we tie the tests our initial, working implementation, we just wrote code we throw away when that implementation changes.

Will these tests increase the confidence in our software? Probably not, because they don't show that the software works. They indicate the presence of recursion.

Wouldn't be better to test that for an integer input, the output is going to be a specific integer? As we learn more about data structures, we might move from recursion to a loop, but our software should continue to calculate the factorial of an integer.

![Codebite loop](/assets/posts/images/2020-02-05/loop.jpg)

Keep in mind the requirements while writing tests, and they will work even after you change the implementation. In our case, the recursion is not a business requirement. The requirement is to calculate the factorial of an integer.

So next time, you're about to test a new feature, think about the moment when someone decides to rewrite your implementation.

What will bring more value to the team?

A test, checking a specific implementation, or a test proving that the problem is solved?
Imagine their face when they run the tests and see all the false positives.

<iframe src="https://giphy.com/embed/6229k5h1JkuvS" width="480" height="358" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/star-trek-trouble-with-tribbles-kirk-gets-shitted-on-by-6229k5h1JkuvS">via GIPHY</a></p>