---
layout:     post
title:      "Removing directory not found warning from Xcode"
date:       2016-03-15 23:02:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   remove-directory-warning
---

I solved this warning removing the following setting: "$(SDKROOT)/Developer/Library/Frameworks"
This options is located in Settings -> Build Settings -> Search Paths -> Framework Search Paths

![Setting location]({{ site.url}}/assets/setting-location.png)

My project continues compiling and working fine, after removing this option.

[http://stackoverflow.com/questions/32040038/warning-directory-not-found-for-option-error-on-build](http://stackoverflow.com/questions/32040038/warning-directory-not-found-for-option-error-on-build/36017148#36017148)