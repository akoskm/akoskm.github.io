---
title: Git Hooks - Deploy with git push
layout: post
type: post
---

Automated deployments are great, simply because you can spend more time developing your app and less time following tedious, repetitive tasks.

They also minimize the human factor of the whole processes so it's more predictable, reliable and error-free.

I assume you already have a bare git repository on the server which looks similar to this:

<pre><code class="hljs text">branches
config
description
HEAD
hooks
info
objects
refs
</code></pre>

We are going to create a `post-receive` hook in the `hooks` directory. This server-side hook is called after the entire push process is completed.

The idea is to pull the contents of a specific branch into a working directory on the server and deploy your application from there. The steps you want to execute after your `post-receive` is triggered are the following:

1. check if we pushed to a specific branch
2. pull the same branch to a server-side working directory
3. start the deployment in the same directory

Here is how our `post-receive` should look like:

<pre><code>#!/bin/bash

# branch to watch for push
DEPLOYMENT_BRANCH=deploy
# server-side working directory
WORKING_DIR=/home/akoskm/deploy/akoskm.github.io

while read oldrev newrev refname
do
    # there can be multiple branches but we're only interested in deploy
    branch=$(git rev-parse --symbolic --abbrev-ref $refname)
    if [ "$DEPLOYMENT_BRANCH" == "$branch" ]; then
      echo "**** pulling changes from $branch ****"

      # change directory to the working copy
      cd $WORKING_DIR
      unset GIT_DIR
      git pull origin $DEPLOYMENT_BRANCH

      # at this point you have the latest source in WORKING_DIR.
      # time to start your build process and deployment scripts
    fi
done
</code></pre>

The script must be executable so don't forget adding

<pre><code class="hljs text">chmod +x post-receive
</code></pre>

Lets test it:

<pre><code class="hljs text">$ git push origin deploy
Counting objects: 7, done.
Writing objects: 100% (3/3), 241 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: **** pulling changes from deploy ****
remote: From /home/akoskm/akoskm.github.io
remote:  * branch            deploy     -> FETCH_HEAD
remote:    ab96f0e..bf21a7f  deploy     -> origin/deploy
remote: Updating ab96f0e..bf21a7f
remote: Fast-forward
remote:  foo | 2 +-
remote:  1 file changed, 1 insertion(+), 1 deletion(-)
To /home/akoskm/akoskm.github.io
   212fc95..bf21a7f  deploy -> deploy
</code></pre>

Pushing to our server triggered the `post-receive` hook and the part specific to `deploy` branch.

Run through this checklist before testing your hook:

+ make sure the server-side working directory exists and you have the required permission to execute your deployment scripts there
+ `post-receive` is executable (`chmod +x`)
+ before pushing check that you're on the right branch (`git branch`)

Sources:

1. [Git on the Server - Getting Git on a Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server)
2. [Customizing Git - Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
