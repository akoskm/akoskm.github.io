---
title: $.find and XML namespaces
layout: post
type: post
---

Querying XML documents
----------------------
It's fine until you face though times and you have to find this:

<pre>
&lt;fancynamespace:tag color="green" size="huge"&gt;Kiwi&lt;/fancynamespace:tag&gt;
</pre>

or a normal-sized Kiwi, that's even harder!

You think, you have a solution
------------------------------

Then you try `$xmlDoc.find('fancynamespace\\:tag')` which [works fine in Firefox but not in Chrome](http://stackoverflow.com/questions/128580/jquery-find-problem). If you decide to ignore the namespace and query only for tag: `$xmlDoc.find('tag')`, that will work in Chrome but not in Firefox.

nodeFilter
----------
[nodeFilter](https://github.com/akoskm/nodeFilter) is a tiny little jQuery plugin which helps you to write selectors for XML elements, even for those with namespaces. It was ispired by these few lines of code:

<pre>
$.fn.filterNode = function(name) {
    return this.find('*').filter(function() {
       return this.nodeName === name;
    });
};
</pre>

[Source](http://stackoverflow.com/questions/853740/jquery-xml-parsing-with-namespaces)

which quickly showed its disadvantage. Because it only considers the nodeName `(fancynamespace:tag)` it will fail even with a slightly complex query: `('fancynamespace:tag[color="red"][size="huge]')`

nodeFilter solves this problem by spliting up the entire query string with this tiny regexp `/(?=\[)/g`, comparing the nodeName with the very first result of split array, then applying the rest of the results to the target element with `$.is`.

I also created a little comparsion table to show, how `$.find` works in different browser (when it comes to namespaces), and what you actually get with nodeFilter:

**Firefox**

|               | `prefix:tag` | `prefix\\:tag` | `tag` | `prefix:tag[color="green"]` |
| ------------- |:------------:|:----------------:|:-----:|:-------------------------------:|
| $.find        |0             |6                 |0      |0                                |
| $.nodeFilter  |6             |0                 |0      |1                                |

<br>

**Chrome**

|               | `prefix:tag` | `prefix\\:tag` | `tag` | `prefix:tag[color="green"]` |
| ------------- |:------------:|:----------------:|:-----:|:-------------------------------:|
| $.find        |0             |0                 |6      |0                                |
| $.nodeFilter  |6             |0                 |0      |1                                |

GitHub
------

[https://github.com/akoskm/nodeFilter](https://github.com/akoskm/nodeFilter)