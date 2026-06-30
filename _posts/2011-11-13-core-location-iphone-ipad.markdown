---
layout:     post
title:      "Core Location iPhone / iPad"
date:       2011-11-13 17:18:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   core-location-iphone-ipad
---

![iPhone location optimizations]({{ site.url}}/assets/tumblr_inline_mjyfyhTr2u1qz4rgp.png)

There are three ways to locate the position of a iPhone device:

1. GPS
2. WiFi
3. 3G

Using the GPS you get the highest accuracy but also it cost more time and use more battery power.

To start using Core Location create and instance of **CLLocationManager** and call **startUpdatingLocation** method. You must implement the **CLLocationManagerDelegate** protocol to start receiving several locations updates with the object **CLLocation**.

Before using **CLLocationManager** you must take care of this three aspects:

1. Specify accuracy you require.
2. Distance filter prevent unneeded calls.
3. Check if location services are enabled.

When an error occurred **didFailWithError** is called.

* **KCLErrorLocationUnknown** is a temporary error that indicates that location cannot be determined.
* **KCLErrorDenied** means that user denied location services for your app.

Best practices:

* Some environments make positioning difficult.
* Call **stopUpdatingPosition** in responds to **KCLErrorLocationUnknown**, and try again later.
* Limit the amount of time that you wait for a location with the accuracy you desire.

When using Map Kit you can set the property **showsUserLocation** to show a blue circle in the location of the device inside the map view.

To use the Core Location in background mode, you must set the variable **UIBackgroundModes** to **location** in the info.plist file.

Optimizations:

* GPS: **KCLLocationAccuracyBest**, **BestForNavigation**.
* GPS: **KCLLocationAccuracyTenMeters**.
* WiFi: **KCLLocationAccuracyHundredMeters**.
* 3G, WiFi: **KCLLocationAccuracyKilometer**, **ThreeKilometers**.

20110324.16