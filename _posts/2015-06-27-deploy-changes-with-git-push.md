---
title: Git Hooks - Deploy with git push
layout: post
type: post
---

Automated deployment processes are great, simply because you can spend more time developing your app and less time following tedious, repetitive tasks.

It also minimizes the human factor which makes your deployments more reliable and error-free. And it's dead simple.

This article will show you how I automated my deployment script with Git Hooks.

Start off by creating a bare git repository on the server:

<pre><code class="hljs text">$ mkdir akoskm.github.io
$ cd akoskm.github.io/
$ git init --bare
Initialized empty Git repository in /home/akoskm/akoskm.github.io/
</code></pre>

If you're unfamiliar with bare git repositories here's a great explanation [What is a bare git repository?](http://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/)
In short, a working git repository is where you will commit changes, while the bare repository only shares those changes between clones (other working repositories).

`git init --bare` will create the following directories:

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

The idea is to pull the contents of a specific branch into a working directory on the server and to deploy your application from there.

Here is how our `post-receive` hook looks like:

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

The script must be executable so don't forget

<pre><code class="hljs text">chmod +x post-receive
</code></pre>

Pushing to our server will trigger the `post-receive` hook:

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

Sources:

1. [Git on the Server - Getting Git on a Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server)
2. [Customizing Git - Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
