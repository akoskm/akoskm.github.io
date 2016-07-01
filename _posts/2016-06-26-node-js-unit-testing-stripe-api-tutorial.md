---
title: Unit Testing Node.js apps and Stripe API integration
layout: post
type: post
date: 2016-07-01 00:00:01
---

Stripe maintains a handful of [official API libraries](https://stripe.com/docs/libraries/),
they are simple and equipped with language-specific examples so you can easily integrate them
into your own software product.

Depending on your application a successful payment has different implications, but it's always a
good idea to make sure that the user gets what he's paid for.

In the Node-Stripe API world this means that the callback which is executed after a payment
has been made behaves correctly in various scenarios. To make sure that it does, let's
cover it with some unit tests.

_This tutorial is about testing your Stripe driven code, if you're still unfamiliar with stripe
I suggest you to read the [Node.js Stripe API Reference](https://stripe.com/docs/api/node#intro)._

Let's say we built an application where the users can order hand-made cosmetics through your web-shop.
Here is the module which handles all the Stripe related stuff:

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

<code>stripe.orders.create</code> returns an [order](https://stripe.com/docs/api#order_return_object)
object if the call succeeded and err otherwise.

Don't forget that we don't want to test the correctness of the Stripe API itself only the behavior of
its callback method. We are going to use [Sinon.JS](http://sinonjs.org) to stub the real Stripe API methods
and return fake responses to our module.

One thing that won't be obvious when you start stubbing Stripe methods is that _create_ isn't
a property of the <code>orders</code> object. That's because
[Orders](https://github.com/stripe/stripe-node/blob/master/lib/resources/Orders.js)
extends
[StripeResource](https://github.com/stripe/stripe-node/blob/master/lib/StripeResource.js) so the order
methods are really in <code>orders.__proto__</code>:

<pre><code class="hljs javascript">
this.sandbox.stub(stripeClient.orders.__proto__, 'create', func);
</code></pre>

The above stub will prevent the real <code>orders.create</code> from being called
and gives you control over triggering its callback with arbitrary parameters.

Let's start by implementing the best-case scenario, when the Stripe API executes our callback with a newly
created order that contains only an id and with no errors:

<pre><code class="hljs javascript">
this.sandbox.stub(stripeClient.orders.__proto__, 'create', function (orderRequest, func) {
  func(null, {id: 123});
});
</code></pre>

Go back to our module we can see what happens in this case:

<pre><code class="hljs javascript">
if (!order || !order.id) {
  cb('Unknown error occurred.', null);
} else {
  cb(null, order);
}
</code></pre>

Since our order was <code>{id: 123}</code> the <code>cb</code> callback passed to <code>createOrder</code>
should be called with <code>null, order</code>.

To prove this just call <code>module.createOrder</code> in your favorite test suit and see if the callback
has the expected parameters:

<pre><code class="hljs javascript">
stripeConnector.createOrder(request, function (err, order) {
  assert.isNull(err);
  assert.isNotNull(order.id);
  assert.equal(123, order.id);
});
</code></pre>

For complete code and running tests the git repository:
[node-stripe-api-testing-tutorial](https://github.com/akoskm/node-stripe-api-testing-tutorial).

Leave a comment below and let me know what you think!