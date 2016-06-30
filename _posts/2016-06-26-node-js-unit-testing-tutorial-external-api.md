---
title: Unit Testing Node.js apps with Stripe API integration
layout: post
type: post
date: 2016-06-25 00:00:01
draft: true
---

Stripe maintains a handful of [official API libraries](https://stripe.com/docs/libraries/),
they are straightforward, equipped with simple, language-specific examples so it's really easy
to integrate it into your own software product.

Depending on your application a successful payment can have different implications, but you always
want to make sure that the user gets what he's paid for.
In the Node-Stripe API world it means that the callback which is executed after the payment behaves
correctly in various scenarios.

_This tutorial is about testing your Stripe driven code, if you're still unfamiliar with stripe
I suggest you to read the [Node.js Stripe API Reference](https://stripe.com/docs/api/node#intro)._

Let's say we built an application where the users can order hand-made cosmetics through your web-shop.
This is the module which handles all the Stripe related stuff:

<pre><code class="hljs javascript">
var stripe = require('stripe')(
  'sk_test_your_stripe_test_key'
);

module.exports = {

  createOrder: function (items, email, cb) {
    // Always validate the user input before handling it to an external API.
    // If you can detect requests that would fail anyway, you can spare the
    // cost of the network request before it's even made.
    if (items === null || items.length < 1) {
      cb('No items found.', null);
    }

    // a terrible way to validate an email address
    if (!email) {
      cb('Email is required.', null);
    }

    stripe.orders.create({
      currency: 'usd',
      items: items,
      email: email
    }, function(err, order) {
      // stripe response, asynchronously called
      if (err) {
        cb('An error occurred.', null);
      } else {
        if (!order || !order.id) {
          cb('Unknown error occurred.', null);
        } else {
          cb(null, order);
        }
      }
    }
  }
});
</code></pre>

The __stripe.orders.create__ returns an [order](https://stripe.com/docs/api#order_return_object)
object if the call succeeded and err otherwise.

Don't forget that we don't want to test the correctness of the Stripe API itself only the behavior of
the callback method. We are going to mock the responses from the Stripe API.
I'm going to use [Sinon.JS](http://sinonjs.org) for this.

One thing that won't be straightforwards when you start stubbing Stripe methods is that _create_ isn't
a property of the orders object. That's because
[Orders](https://github.com/stripe/stripe-node/blob/master/lib/resources/Orders.js)
extends a base class called
[StripeResource](https://github.com/stripe/stripe-node/blob/master/lib/StripeResource.js) so the order
methods are really in <code>orders.__proto__</code>:

<pre><code class="hljs javascript">
this.sandbox.stub(stripeClient.orders.__proto__, 'create', cb);
</code></pre>

The above stub will prevent the real <code>orders.create</code> from being called
and will call <code>cb</code> instead. But what <code>cb</code> represents? <code>cb</code> is the callback
in your module containing the <code>createOrder</code> method:
<pre><code class="hljs javascript">
stripe.orders.create({
  currency: 'usd',
  items: items,
  email: email
}, function(err, order) { // &lt;- this is cb
  // left for brevity
}
</code></pre>
Let's start by implementing the best-case scenario, when the API executes our callback without errors and with
a newly created order that contains an id:

<pre><code class="hljs javascript">
this.sandbox.stub(stripeClient.orders.__proto__, 'create', function (orderRequest, cb) {
  cb(null, {id: 123});
});
</code></pre>

If we go back to our module we can see what happens in such case:

<pre><code class="hljs javascript">
if (!order || !order.id) {
  cb('Unknown error occurred.', null);
} else {
  cb(null, order);
}
</code></pre>

Since our order was <code>{id: 123}</code> the callback of <code>createOrder</code> should be called with
<code>null, order</code>.

In order to prove this we'll call <code>module.createOrder</code> in our test suit and see if <code>cb</code> really
carries the parameters we expect:

<pre><code class="hljs javascript">
stripeConnector.createOrder(request, function (err, order) {
  assert.isNull(err);
  assert.isNotNull(order.id);
  assert.equal(123, order.id);
});
</code></pre>

For complete code and running tests the git repository:
[node-stripe-api-testing-tutorial](https://github.com/akoskm/node-stripe-api-testing-tutorial).