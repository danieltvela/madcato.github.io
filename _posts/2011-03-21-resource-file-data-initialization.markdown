---
layout:     post
title:      "Resource file for data initialization"
date:       2011-03-21 22:24:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   resource-file-data-initialization
---

To initialize a NSArray with sample data or configuration data, you can store a data file in the resources of the xcode project like shown in figure:

![resource file xcode]({{ site.url }}/assets/tumblr_inline_mjul4okGO51qz4rgp.png)

To retrive data use the following code:

{% highlight objc %}
// Load the data of file: "Data.plist".  
NSString *dataPath = [[NSBundle mainBundle] pathForResource:@"Data" ofType:@"plist"];  
self.data = [NSArray arrayWithContentsOfFile:dataPath];
{% endhighlight %}

*\* The array representation in the file identified by aPath must contain only property list objects (NSString, NSData, NSDate, NSNumber, NSArray, or NSDictionary objects)*