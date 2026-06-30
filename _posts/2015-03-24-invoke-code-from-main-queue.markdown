---
layout:     post
title:      "Invoke code from the main queue"
date:       2015-03-24 12:13:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   invoke-code-from-main-queue
---


To invoke some code from the main queue thread in iOS use the following code:

{% highlight objc %}
dispatch_async(dispatch_get_main_queue(), ^{ /* some code like show an alert view */ });
{% endhighlight %}