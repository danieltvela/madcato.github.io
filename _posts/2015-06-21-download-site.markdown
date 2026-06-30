---
layout:     post
title:      "Download a site"
date:       2015-06-21 19:42:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   download-site
---


On Mac OS X open Terminal and type:

{% highlight bash %}
$ wget --page-requisites --convert-links http://URL-to-Start
{% endhighlight %}

However, this will only download those files that are referenced from the entry URL, so you might need to run it on all sub-URLs individually.