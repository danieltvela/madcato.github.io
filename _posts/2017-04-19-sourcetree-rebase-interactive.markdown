---
layout:     post
title:      "How to make a rebase with Source Tree creating only one new commit with new message"
date:       2017-04-19 01:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   sourcetree-rebase-interactive
---


If you want to merge two branches- but don't want to keep separated banches and fix the messages of branch-, you can use an **interactive rebase**. 

Using **rebase**, branches aren't maintained in the repository. Using rebase you can move one branch to the top of another.

![original]({{ site.url}}/assets/original.png) ![rebase]({{ site.url}}/assets/rebase.png)

But **git-rebase** has another fantastic feature: **interactive rebase**. With this type of **rebase** you can edit commit messages, even mix several commits into one. This is very useful to keep a well documented **git log** without noise.

![feature]({{ site.url}}/assets/feature.png)

To use this method, first checkout the branch you want to rebase(the secondary branch). Then select the last commit of destination branch(the main one). 

![interactive]({{ site.url}}/assets/interactive.png)

**To avoid merge conflicts is very, very important to apply command "Squash with previous commit..." always on top of the commit list after rebase. After all the *Squash* the commits must by ordered**

![squash]({{ site.url}}/assets/squash.png)

## Final result

Only one thread, only one commit message. Cheers!!!

![final]({{ site.url}}/assets/final.png)

[Additional info from atlassian web](https://www.atlassian.com/blog/sourcetree/interactive-rebase-sourcetree)

*In git we trust*
