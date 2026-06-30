---
layout:     post
title:      "How to run Swift from Command Line"
date:       2015-06-16 17:28:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   run-swift-command-line
---


To run a Swift script in Terminal type the following: 

{% highlight swift %}
$ swift fileName.swift
{% endhighlight %}

Alternatively you can put the following line of text at the top of the script file and run that file

{% highlight swift %}
#!/usr/bin/env xcrun --sdk swift
{% endhighlight %}