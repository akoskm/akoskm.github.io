---
title: Implementing Sign in with Twitter
layout: post
type: post
---
<!--If you already succeed implementing a social sign-in process in Java using popular libraries like [scribe-java](https://github.com/fernandezpablo85/scribe-java) or [twitter4j](https://github.com/yusuke/twitter4j/), then congratulations. But you forgot to report an ambiguity which is present in both libraries.
-->
In [#560](https://github.com/fernandezpablo85/scribe-java/issues/560) I had a discussion with [@fernandezpablo][3] about the [scribe-java][4] API. You already have couple of examples for [pin-based](https://dev.twitter.com/oauth/pin-based) authorization, which is fine in desktop apps. However, when it comes to web applications you probably want something else.

With the current API it isn't straightforward how to implement a sign-in process, which does not require PIN authorization from the user but uses the "Authorize this app to use your account" page to request access to the user's twitter profile.

![Authorize Twitpic](https://si0.twimg.com/help/1343078567_71751)

Both scribe-java ([TwitterExample.java][1]) and twitter4j ([Code Examples][2], skip to _7. OAuth support_) uses these three steps to obtain a new access token:

1. Obtain a request token object
2. Authorize the 3rd party application by entering a PIN, this generates you a Verifier object (no Verifier object in twitter4j, the pin is used directly in the next step as verifier)
3. Convert the  previously obtained request token and verifier to an access token

The workflow for obtaining an access token without entering a PIN looks slightly different:

1. Obtain a request token object, and define a redirect URL. This will be required in step 3.
2. Redirect the user to the authorization URL contained in the request token object received in step 1. This URL leads to the "Authorize this app to use your account" page
3. User clicks Sign In
3. Twitter approves the authorization request and the response is a redirect to the URL supplied in step 1. with the following query parameters: **oauth_token**, **oauth_verifier**.
4. At this step both scribe-java and twitter4j expecting you to supply a request token object in order to obtain an access token. According to [Step 3: Converting the request token to an access token](https://dev.twitter.com/web/sign-in/implementing), everything you need to obtain a request token are **oauth_token** and **oauth_verifier** received in the previous redirect request.

At this point you have what Twitter asked for but you still can't figure out the next step looking at the API.

You can manually create a new `Token` by using **oauth_token** as token and **oauth_verifier** as secret in:

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

and a new `Verifier` from **oauth_verifier**:

<pre><code class="hljs java">
Token requestToken = new Token(oauthToken, oauthVerifier);
Verifier verifier = new Verifier(oauthVerifier);
</code></pre>

Finally, retrieve a new access token from Twitter:
<pre><code class="hljs java">
// POST oauth/access_token
Token accessToken = service.getAccessToken(requestToken, verifier);

// Do something with the access token
OAuthRequest request = new OAuthRequest(Verb.GET, PROTECTED_RESOURCE_URL);
service.signRequest(accessToken, request);
</code></pre>

I've set up a little demo project showing these steps in action:

[twitter-sign-in-example](https://github.com/akoskm/twitter-sign-in-example)

If you want to discuss leave a comment below or hit me up on [@akoskm](https://twitter.com/akoskm).

[1]: https://github.com/fernandezpablo85/scribe-java/blob/master/src/test/java/org/scribe/examples/TwitterExample.java
[2]: http://twitter4j.org/en/code-examples.html
[3]: https://twitter.com/fernandezpablo
[4]: https://github.com/fernandezpablo85/scribe-java
