---
title: Browserify Boilerplate
layout: post
type: post
date: 2016-09-18 00:00:01
---

Chances that you can set up a project from scratch and use whatever technologies
you're familiar with doesn't happen every day. Sometimes you just have to work with what you've got.
It's been a while since we started tweaking [blueprint3d](https://github.com/furnishup/blueprint3d) -
an open source project for designing interior spaces such as homes or apartments.
If you didn't hear about it yet, check out the demo:
[http://furnishup.github.io/blueprint3d/example/](http://furnishup.github.io/blueprint3d/example/)!

### What we had

Being an open source project, blueprint3d contains a single npm script which builds the application without
any further minification or obfuscation, it's pretty simple:

<pre><code>  "scripts": {
    "build": "browserify src/blueprint3d.js > example/js/blueprint3d.js --verbose"
  },
</code></pre>

### minifyify

We wanted at least to compress the final file, if possible, obfuscate it. Browserify + minify + uglify, that's
how we came to [minifyify](https://www.npmjs.com/package/minifyify).