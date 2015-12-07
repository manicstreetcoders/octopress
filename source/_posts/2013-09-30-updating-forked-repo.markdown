---
layout: post
title: "Updating Forked Repo"
date: 2013-09-30 13:16
comments: true
categories: 
---
[How to update github forked repository](http://stackoverflow.com/questions/7244321/how-to-update-github-forked-repository)

    git remote add upstream git://github.com/whoever/whatever.git

    # Fetch all the branches of that remote into remote-tracking branches,
    # such as upstream/master
    git fetch upstream

    # Make sure that you're on your master branch:
    git checkout master

    # Rewrite your master branch so that any commits of yours that
    # aren't already in upstream/master are replayed on top of that
    # other branch:
    git rebase upstream/master

    git push -f origin master

