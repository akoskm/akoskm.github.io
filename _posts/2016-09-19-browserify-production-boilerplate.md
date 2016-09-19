---
title: Browserify Boilerplate
layout: post
type: post
date: 2016-09-18 00:00:01
---

Chances that you can set up a project from scratch and use whatever technologies
you're familiar with doesn't happen every day. Sometimes you just have to work with what you've got.
It's been a while since we started working on a project that is based on [blueprint3d](https://github.com/furnishup/blueprint3d) -
an open source project for designing interior spaces such as homes or apartments.
If you didn't hear about it yet, check out their demo:
[http://furnishup.github.io/blueprint3d/example/](http://furnishup.github.io/blueprint3d/example/)!

### Initial setup

blueprint3d, being an open source project, contains a single npm script which builds the application without
any further minification or obfuscation, it's pretty simple:

<pre><code>"scripts": {
  "build": "browserify src/blueprint3d.js > example/js/blueprint3d.js --verbose"
}</code></pre>

### Goals

The previous command dumps everything into a single file, which is a bit uncomfortable for
debugging. You can't just jump to specific line in a specific file since everything is contained in
<code>blueprint3d.js</code>. So we wanted somehow to make debugging easier while having a compressed,
if possible obfuscated, final file in production.

### Mistake 1 - not checking what you're existing tool is capable of

Browserify + minify + uglify, that's
how we came to [minifyify](https://www.npmjs.com/package/minifyify). Thanks to minifyify's simplistic github page
the command what we needed was right there, at least we thought:

<pre><code>$ browserify entry.js -d -p [minifyify --map bundle.js.map --output bundle.js.map --uglify [ --compress [ --dead_code--comparisons 0 ] ] ] > bundle.js</code></pre>

Sourcemaps: checked, uglify: checked. All set, let's give it a try!
We indeed could open individual files for debugging, only the debugger wasn't stepping over the source as we expected.

> If you are currently using uglifyify and realized that your sourcemap is behaving strangely, you're not alone. 

This is one of the [advantages of minifyify](https://github.com/ben-ng/minifyify#advantages) and we weren't using uglifyify
at all.

### Mistake 2 - instead of rethinking your approach digging deeper

All right, so there was a tool uglifyify which was mentioned earlier so we went straight with uglifyify.
It behaved pretty much the same like minifyify did. At this point we had no choices then to browse through minifyify
issues and try to figure out if someone had the same problem.

Reading [this](https://github.com/ben-ng/minifyify/issues/64#issuecomment-54495037) comment we realized that
minifyify was obviously made for production and we're thinking in the wrong direction.
If minifyify is meant for production and my problem is with having to debug everything inside a single file,
then the problem is probably with browserify?

### -d

[Browserify usage](https://github.com/substack/node-browserify#usage):

<pre>
 --debug -d  Enable source maps that allow you to debug your files

             separately.
</pre>

Adding this single flag to the existing npm script:

<pre><code>"scripts": {
  "build": "browserify -d src/blueprint3d.js > example/js/blueprint3d.js --verbose"
}
</code></pre>

made the original source files debuggable, without any additional plugin.

### Production build

Here is how our production script looked like:

<pre><code>"scripts": {
  "build": "browserify -d src/blueprint3d.js > example/js/blueprint3d.js --verbose"
  "production": "browserify src/blueprint3d.js -p [minifyify --no-map] > example/js/blueprint3d.js"
}</code></pre>

In the end we went with minifyify, seems that the [2nd advantage](https://github.com/ben-ng/minifyify#advantages)
of minifyify about creating Smaller Bundles then uglify-js itself seems to be true. With uglify-js our final bundle
was 811 KB while with minifyify only 596 KB.