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

Then you try `$xmlDoc.find('fancynamespace\\:tag')` which [works fine in FireFox but not in Chrome](http://stackoverflow.com/questions/128580/jquery-find-problem). If you decide to ignore the namespace and query only for tag: `$xmlDoc.find('tag')`, that will work in Webkit but not in FireFox.

nodeFilter
----------
nodeFilter is a tiny little jQuery plugin which helps you to write selector for you XML elements, even for those with namespaces. It was ispired by these few lines of code:

<pre>
$.fn.filterNode = function(name) {
    return this.find('*').filter(function() {
       return this.nodeName === name;
    });
};
</pre>

[Source](http://stackoverflow.com/questions/853740/jquery-xml-parsing-with-namespaces)

which quickly showed its disadvantage. Because it only considers the nodeName `(fancynamespace:tag)` you cannot write a more custom query like: `('fancynamespace:tag[color="red"][size="huge]')`

nodeFilter solves this problem by spliting up the entire query string with this tiny regexp `/(?=\[)/g`, comparing the nodeName with the very first result of split, then applying the rest of split to the element node with `$.is`.
