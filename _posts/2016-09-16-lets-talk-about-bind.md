---
title: Lets talk about bind
layout: post
type: post
date: 2016-09-16 00:00:01
---

A few days ago I was asked if I could explain the difference between apply call and bind.
Both methods used to call functions. They're expecting _this_ as their first argument, and if you ever read this clever comparison on [stackoverflow](http://stackoverflow.com/questions/1986896/what-is-the-difference-between-call-and-apply#comment14359320_1986896):

> Think of a in apply for array of args and c in call for columns of args.

you'll hardly ever forgets what's the only real difference.
To make things clear: apply expects the arguments after this in array-like form while call expects them individually.

The only thing I knew about bind is that it annoyed the hell out of me.
If you did any React programming you know that there is [No Autobinding](https://facebook.github.io/react/docs/reusable-components.html#no-autobinding)
and you have to explicitly use .bind(this) on every custom function of your React component (or as an alternative you can use [=> functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)).

So I was using _bind_ regularly, I know why I'm using it, but I had no idea how it compares to apply and call.
Which obviously demands a research and some examples! Here is what I found out:

### Similarities

1. All three function accept this as their first argument. _bind_ is more _call_-like meaning that it expects the arguments individually.
This doesn't mean that you can't pass an array to it but the result might surprise you, more about this later.

### Differences

1. <code>apply</code> and <code>call</code> used to invoke functions, <code>bind</code> _creates_ a new function.

<pre><code class="hljs javascript">var cat = {
  name: 'Milu',
  meow: function (greeting) {
    return greeting + ', ' + this.name;
  }
};

cat.meow.apply(cat, ['Hi']); // returns Hi, Milu
cat.meow.call(cat, 'Hello'); // returns Hello, Milu
var meowBind = cat.meow.bind(cat, 'Hey');   // returns a function
meowBind() // returns Hey, Milu
</code></pre>

These things can be found in the documentation so now let's do something what's isn't there.

### Invocation with a function instead of an object

Depending on the implementation of the function which is binded, it may have interesting results.
Since functions possess a name property the following call will result in:
<pre><code class="hljs javascript">function purr() {
  return 'purrs at a frequency of 20 to 30 vibrations per second';
}

cat.meow.apply(purr, ['Hey']); // returns Hey, purr
</code></pre>

The same thing will happen when you call <code>apply</code> or <code>call</code> while providing <code>purr</code> as first argument.

### Invocation with an array of arguments

Remember how <code>apply</code> works, it takes arguments in array-like format:

<pre><code>var cat = {
  name: 'Milu',
  meow: function (greeting) {
    return greeting + ', ' + this.name;
  }
};

cat.meow.apply(cat, ['Hi', 'Hello']); // returns Hi, Milu
</code></pre>

Since <code>meow</code> takes only one arguments, the rest of the array is ignored.
If it would take more parameters it would look like:
<pre><code>meow: function (greeting, greeting2) {
  return greeting + ', ' + greeting2 + ', ' + this.name;
}</code></pre>
and output would be <code>Hi, Hello, Milu</code>.

Even if <code>bind</code> accepts an array as its second argument it
handles that array completely different from <code>apply</code>:

<pre><code>var cat = {
  name: 'Milu',
  meow: function (greeting) {
    return greeting + ', ' + this.name;
  }
};

cat.meow.bind(cat, ['Hi', 'Hello']); // returns Hi,Hello, Milu
</code></pre>

<code>bind</code> converst the array with <code>toString</code> then passes it to the newly created function.