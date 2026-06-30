---
layout:     post
title:      "Launching the App Store from iPhone app"
date:       2011-02-15 16:13:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   launching-app-store
---

![itunes app icon]({{ site.url}}/assets/tumblr_inline_mjpcxshVrk1qz4rgp.png)

1. Launch iTunes on your computer.
2. Search for the item you want to link to.
3. Right-click or control-click on the item’s name in iTunes, then choose “Copy iTunes Store URL” from the pop-up menu.
4. Open the modified URL using an NSURL object and the -[UIApplication openURL] method.

{% highlight objc %}
// Watch Technical Q&amp;A QA1629  
// Launching the App Store from an iPhone application  
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"http://itunes.apple.com/es/app/lishop/id415119777?mt=8"]];  
{% endhighlight %}

For url format see [App Store Marketing](https://developer.apple.com/appstore/resources/marketing/)