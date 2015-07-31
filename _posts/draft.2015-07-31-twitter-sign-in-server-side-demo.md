---
title: Implementing Sign in with Twitter - Part I
layout: post
type: post
---
Social sign-in saves your new users from the painful and repetitive task of filling out _another_ registration. I have to be very intrigued to sign-up for an application without social login.

If you already succeed implementing a social sign-in process in Java using popular libraries like [scribe-java](https://github.com/fernandezpablo85/scribe-java) or [twitter4j](https://github.com/yusuke/twitter4j/), then congratulations. But you forgot to report an ambiguity which is present in both libraries.

In [#560](https://github.com/fernandezpablo85/scribe-java/issues/560) I tried to point out what's wrong in both these libraries. Look at how scribe is obtaining an access token (based on [TwitterExample.java](https://github.com/fernandezpablo85/scribe-java/blob/master/src/test/java/org/scribe/examples/TwitterExample.java)):

1. Obtain a request token object
2. Authorize the 3rd party application by entering a PIN, this generates you a Verifier object
3. Obtain an access token from the previously generated request token and verifier
4. Do something with the access token

twitter4j (based on [Code Examples](http://twitter4j.org/en/code-examples.html), skip to _7. OAuth support_):

1. Obtain a request token object
2. Obtain an access token from the previously generated request token and the PIN entered.
3. Do something with the access token

twitter4j has one object less but they are using the same process for obtaining the access token. They are requiring you to enter a PIN.

Wait, this is how social sign-in supposed to work? Shouldn't I see a dialog like this:

![Authorize Facebook](http://i1.wp.com/techverse.net/wp-content/uploads/2013/10/authorize-facebook-to-access-twitter-and-share-updates.jpg)

Will any of the above examples result in this? No.

The workflow for obtaining an access token without entering a PIN should be something like this:

1. Obtain a request token object, but don't forget to supply a redirect URL. This will be required in step 3.
2. Redirect the user to the authorization URL contained in the request token object received in step 1. This URL leads to the "Authorize -you application name- to use your account" dialog you're already familiar with.
3. When Twitter approves you authorization request, the response is going to be redirect to the URL supplied in step 1. with the following query parameters **oauth_token** and **oauth_verifier**.
4. At this step both scribe-java and twitter4j expecting you to supply a request token object in order to obtain an access token. According to [Step 3: Converting the request token to an access token](https://dev.twitter.com/web/sign-in/implementing), the only necessary things to obtain a request token are **oauth_token** and **oauth_verifier** received in the previous redirect request.

Basically you reached the point when you have what Twitter asked for but you still can't figure out the next step looking at the API.

All you have is the **oauth_token** and the **oauth_verifier** strings and the ability to construct a service object. So how to obtain a request token and a verifier?

The API doesn't tell you is that you can use the verifier **oauth_verifier** as secret in:

<pre><code class="hljs java">
/**
 * Default constructor
 *
 * @param token token value. Can't be null.
 * @param secret token secret. Can't be null.
 */
public Token(String token, String secret)
{
  this(token, secret, null);
}
</code></pre>

Therefore, a request token and the verifier object can be reconstructed from the **oauth_token** and **oauth_verifier** strings like this:

<pre><code class="hljs java">
Token requestToken = new Token(oauthToken, oauthVerifier);
Verifier verifier = new Verifier(oauthVerifier);
</code></pre>

At this point retrieving the access token object becomes trivial:
<pre><code class="hljs java">
// POST oauth/access_token
Token accessToken = service.getAccessToken(requestToken, verifier);

// Do something with the access token
OAuthRequest request = new OAuthRequest(Verb.GET, PROTECTED_RESOURCE_URL);
service.signRequest(accessToken, request);
</code></pre>

I've set up a little demo project showing these steps in action:

[twitter-sign-in-example](https://github.com/akoskm/twitter-sign-in-example)

If you have any ideas, comments, suggestions, leave a comment or hit me up on [@akoskm](https://twitter.com/akoskm).
