---
layout:     post
title:      "Debug with Objective-C"
date:       2012-04-08 11:24:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   debug-objectivec
---


## Preprocessor macros for filename, line number and function

{% highlight objc %}
NSLog(@"\n Function: %s\n Pretty function: %s\n Line: %d\n File: %s\n Object: %@", __func__, __PRETTY_FUNCTION__, __LINE__, __FILE__, button);
{% endhighlight %}

## Aditional information

{% highlight objc %}
NSLog(@"Current selector: %@", NSStringFromSelector(_cmd));
NSLog(@"Object class: %@", NSStringFromClass([self class]));
NSLog(@"Filename: %@", [[NSString stringWithUTF8String:__FILE__] lasPathComponent]);
{% endhighlight %}

## Call stack

{% highlight objc %}
NSLog(@"Stack trace: %@", [NSThread callStackSymbols]);
{% endhighlight %}

## In Xcode console write

{% highlight objc %}
po [NSThread callStackSymbols]
{% endhighlight %}

## How to add debug information to a Release version

In buil settings search for **“Generate Debug Symbols”** and set the checkbox. Also search for **“Level of Debug Symbols”** set **“All symbols”**

## If Relase build crashes but Debug one not, check the following

Disable **“Optimizations”** in build setting for release build. Set **“None”**. Check this link: How to Fix an [App that Crashes in Release but not Debug](http://www.mindjuice.net/2011/11/30/how-to-fix-an-app-that-crashes-in-release-but-not-debug/)

20110718.34

