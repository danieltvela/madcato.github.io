---
layout:     post
title:      "Background iOS location"
date:       2011-11-19 18:10:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   background-ios-location
---

![gps]({{ site.url}}/assets/tumblr_inline_mk04xflxbT1qz4rgp.png)

To operate the location of the iPhone in any way **background** should use the following:

1. Activate the file project.plist the next value: **UIBackgroundModes**: Array => **location**
2. Implement an CLLocationManagerDelegate intarface and enable tracing with one of two methods:
	* **startMonitoringSignificantLocationChanges**
The problem with this method is that you throw few updates: only when there is a change in cell mobile telephony. The good thing is that it consumes little battery power.
	* **startUpdatingLocation**
By using this method is very important correctly configure the following variables:
		* **desiredAccuracy** => precision
		* **distanceFilter** => Setting **kCLDistanceFilterNone** will be received position updates. Not recommended, unless high accuracy is required.


To avoid using too much battery life is better **desiredAccuracy** put a larger one. For example 100 m. for displaced receive updates every 100 m.

20110324.42

