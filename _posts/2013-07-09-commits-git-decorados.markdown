---
layout:     post
title:      "Decorated git commits"
date:       2013-07-09 20:27:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   commits-git-decorados
---


In order to see the commits of a repository in a cool way.

{% highlight bash %}
git log --graph --decorate --pretty=oneline --abbrev-commit --all
{% endhighlight %}