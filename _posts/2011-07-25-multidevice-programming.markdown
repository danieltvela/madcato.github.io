---
layout:     post
title:      "Multidevice programming"
date:       2011-07-25 19:34:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   multidevice-programming
---

![xcode icon]({{ site.url }}/assets/tumblr_inline_mjwwlnUiXL1qz4rgp.jpg)

To create a project from another project iPad iPhone, Xcode have a choice in a popover that lets you:

* Create a universal project for iPhone and iPhone.
* Create two different targets, one for iPhone and another iPhone

## UI Idiom

{% highlight objc %}
UIDevice.h
typedef enum {
UIUserInterfaceIdiomPhone,
UIUserInterfaceIdiomPad
} UIUserInterfaceIdiom;

#define UI_USER_INTERFACE_IDIOM()...

// Sample code
if(UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) {
... // iPad code
} else {
... // iPhone code
}
{% endhighlight %}

20110324.14