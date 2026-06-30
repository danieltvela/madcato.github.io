---
layout:     post
title:      "iOS Multiple Version Programming"
date:       2011-08-07 10:55:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   ios-multiple-version-programming
---

To use classes in a later version of our SDK “Target version” use this code:

{% highlight objc %}
Class myClass = NSClassFromString(@"UILocalNotification");  
if(myClass) {  
    UILocalNotification* alarm = [[myClass alloc] init];  
}  
{% endhighlight %}
For methods use:

{% highlight objc %}
UIDevice* device = [UIDevice currentDevice];  
BOOL multitaskingSupported = NO;  
if([device respondsToSelector:@selector(isMultitaskingSupported)]) {  
    multitaskingSupported = [device isMultitaskingSupported];  
}  
{% endhighlight %}

For functions use:

{% highlight objc %}
 if(&UIGraphicsBeginPDFContextToFile != NULL) {  
      UIGraphicsBeginPDFContextToFile(...);  
 }  
 {% endhighlight %}
 
The same can be used for constant symbols.

For using later version frameworks, configure it as “weak-linking” in target build settings.

20110324.27

