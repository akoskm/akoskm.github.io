---
title: Let's talk about bind
layout: post
type: post
date: 2016-09-16 00:00:01
redirect_from:
 - /2016/09/16/lets-talk-about-bind.html
---

A few days ago I was asked if I could tell the difference between <code>apply</code>, <code>call</code> and <code>bind</code>.
Both <code>apply</code> and <code>call</code> used to call functions and there are dozens of posts explaining how they do that.
They're expecting <code>this</code> as their first argument, and if you ever read this clever comparison on [Stack Overflow](http://stackoverflow.com/questions/1986896/what-is-the-difference-between-call-and-apply#comment14359320_1986896):

> Think of a in apply for array of args and c in call for columns of args.

you'll hardly ever forget what's the only real difference.
<code>apply</code> expects the arguments after <code>this</code> in array-like form while <code>call</code> expects them individually.

Because of [No Autobinding](https://facebook.github.io/react/docs/reusable-components.html#no-autobinding) in React
I used <code>bind</code> regularly, but I still had no idea how it compared to <code>apply</code> or <code>call</code>, so here are my findings:

### Similarities

1. All three functions accept <code>this</code> as their first argument. <code>bind</code> is more
<code>call</code>-like meaning that it expects the arguments to be passed individually.

### Differences

1. <code>apply</code> and <code>call</code> used to invoke functions, <code>bind</code> _creates_ a new function.
When you invoke this function it will has its <code>this</code> keyword set to the value you provided as first argument.

```javascript
var cat = {
  name: 'Milu',
  greet: function (greeting) {
    return greeting + ', ' + this.name;
  }
};

cat.greet.apply(cat, ['Hi']); // returns Hi, Milu
cat.greet.call(cat, 'Hello'); // returns Hello, Milu
var greetBound = cat.greet.bind(cat, 'Hey');   // returns a function
greetBound() // returns Hey, Milu
```

These things can be found in the documentation so let's do something what isn't there.

### Invocation with Function objects

Below is a function `purr`, which in fact is a <code>Function</code> object. Every <code>Function</code> object has a
<code>name</code> property so the following call will result in:

```javascript
function purr() {
  return 'purrs at a frequency of 20 to 30 vibrations per second';
}

var greetBound = cat.greet.bind(purr, ['Hey']);
greetBound(); // returns Hey, purr
```

The same thing will happen when you call <code>apply</code> or <code>call</code> while providing <code>purr</code> as first argument.

### Invocation with an array of arguments

Passing an array of arguments to <code>bind</code> then calling the bound function will have the same effect like
passing it to <code>call</code>:

```javascript
var cat = {
  name: 'Milu',
  greet: function (greeting) {
    return greeting + ', ' + this.name;
  }
};

cat.greet.call(cat, ['Hi', 'Hello']); // returns Hi,Hello, Milu
var greetBound = cat.greet.bind(cat, ['Hi', 'Hello']);
greetBound(); // returns Hi,Hello, Milu
```

<code>bind</code> converts its second argument to string before passing it to the bound function.

### When to use bind

Bind is really just a <code>call</code> except that the invocation of the bound function can be delayed.
This reminds us to callbacks functions. Sadly, most of the examples I encountered did use <code>bind</code>
with callback functions but they didn't had the most thoughtful design,
which justified the usage of <code>bind</code>:

```javascript
var Widget = function () {
  this.counter = 0;
  $('button').on('click', function () {
    this.counter += 1;
    $('span').text(this.counter);
  }.bind(this));
};
```

A simpler widget:

```javascript
var Widget = function () {
  var counter = 0;
  $('button').on('click', function () {
    counter += 1;
    $('span').text(counter);
  });
};
```

Browsing through dozens of examples I saw that <code>bind</code> is more extensively used where the code
has conventional object-oriented design - which I've been avoiding altogether when it comes to JavaScript, and it pretty
much explains why I had no idea how it compares to <code>call</code> and <code>apply</code>.

Do you have a specific scenario where <code>bind</code> is inevitable, or more appropriate? Let me know in the comments!