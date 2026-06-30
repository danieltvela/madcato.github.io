---
layout:     post
title:      "Objective-C helpers"
date:       2011-12-04 11:24:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   objectivec-helpers
---

## Blocks

A block of code is an structure with this format:

{% highlight objc %}
^ <return_type> <parameters> {<code>}   
{% endhighlight %}

like:
{% highlight objc %}
^BOOL(id str){return [str length] > num;}  
^{[local doSomething];}  
{% endhighlight %}
This structures can be assigned or passed as a parameter or stacked.

## Iterations

{% highlight objc %}
NSArray* array;  
for(id e in array) {...e...}  
NSDictionary* dictionary;  
for(id k in dictionary){...k...}  
[array enumerateObjectsWithOptions:NSEnumerationReverse   
                        usingBlock:^(id e,NSUInteger idx,BOOL* stop){...e...idx}];  
[dictionary enumerateKeysAndObjectsUsingBlock:  
         ^(id k,id obj,BOOL* stop){...k...obj;}]  
{% endhighlight %}

## Memory management

When you have two objects that have a reference to each other, use this convention to avoid release locks: Downlinks member instance use retain. Uplinks member instance don’t use retain.

This avoid memory locks like this one:

![obejcts never released]({{ site.url}}/assets/tumblr_inline_mk05bvcs3R1qz4rgp.png)
