---
title: Enabling custom domain for your heroku app with HTTPS
layout: post
type: post
date: 2019-09-03 00:00:01
---

Step 1.

Buy a domain if you don't have one.

Step 2.

Log in to your heroku console:

```
$ heroku login
heroku: Press any key to open up the browser to login or q to exit:
Opening browser to https://cli-auth.heroku.com/auth/browser/f2737dbd-d14c-11c0-bafd-f12ff3d2499d
Logging in... done
Logged in as hello@akoskm.com
```

Step 3.

Assign the custom domain to your heroku app:

```
$ heroku domains:add -a my-heroku-app my-heroku-app.akoskm.com
Adding my-heroku-app.akoskm.com to ⬢ my-heroku-app... done
 ▸    Configure your app's DNS provider to point to the DNS Target cute-cats-puivs2izaopsep1mcu2ae3m8li.herokudns.com.
 ▸    For help, see https://devcenter.heroku.com/articles/custom-domains

The domain my-heroku-app.akoskm.com has been enqueued for addition
 ▸    Run heroku domains:wait 'my-heroku-app.akoskm.com' to wait for completion
```

Step 4.

Go to the DNS zone editor of your domain and add a CNAME entry. The naming in your DNS editor might be different but here are the only a few things you have to fill:

```
Zone name: my-heroku-app.akoskm.com
Fully qualified domain name: cute-cats-puivs2izaopsep1mcu2ae3m8li.herokudns.com. <- should be the DNS Target heroku domains:add gave you
Type: CNAME (this is usually prefilled if your DNS editor is smart enough)
TTL: 14400 (mine says 14400, that's the default)
```

This is how it looks like in CPanel:

![cname editor](https://i.imgur.com/uHe8hze.png)

At this point you should be able to navigate to http://my-heroku-app.akoskm.com and see your heroku app.
`https://` access shouldn't work at this point.

Step 5.

Adding SSL


You can check if auto certifcate management is enabled for your app:

```
$ heroku certs:auto -a my-heroku-app
=== Automatic Certificate Management is disabled on my-heroku-app
```

if it says disabled simple enable it:

```
$ heroku certs:auto:enable -a my-heroku-app
Enabling Automatic Certificate Management... starting. See status with heroku certs:auto or wait until active with heroku certs:auto:wait
=== Your certificate will now be managed by Heroku.  Check the status by running heroku certs:auto.
```

When you check the status it should display something like this:
```
$ heroku certs:auto -a my-heroku-app
=== Automatic Certificate Management is enabled on my-heroku-app

Certificate details:
Common Name(s): my-heroku-app.akoskm.com
Expires At:     2019-12-09 07:19 UTC
Issuer:         /C=US/O=Let's Encrypt/CN=Let's Encrypt Authority X3
Starts At:      2019-09-10 07:19 UTC
Subject:        /CN=my-heroku-app.akoskm.com
SSL certificate is verified by a root authority.

Domain              Status       Last Updated
──────────────────  ───────────  ──────────────────
my-heroku-app.akoskm.com  Cert issued  less than a minute
```


Source:
 - [Heroku Dev Center: Custom Domain Names for Apps](https://devcenter.heroku.com/articles/custom-domains)
 - [Heroku Dev Center: Automated Certificate Management](https://devcenter.heroku.com/articles/automated-certificate-management)