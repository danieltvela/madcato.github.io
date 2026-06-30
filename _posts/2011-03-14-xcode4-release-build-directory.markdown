---
layout:     post
title:      "Xcode 4: release build directory"
date:       2011-03-14 11:19:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   xcode4-release-build-directory
---

In Xcode 4 the default preferences specify a build output directory different from de Xcode 3 one. You can change this configuration and let the Xcode works as before. Open Xcode 4 and go to Preferences -> Locations and change “Build location”.

![xcode setting]({{ site.url }}/assets/tumblr_inline_mjukw5hSwY1qz4rgp.png)

If you want to use the current recommended configuration, you must know how to build and find the release binary: Build the project with option “Product -> Build For -> Build for Archiving” Then go to path *~/Library/Developer/Xcode/DerivedData//Build/Products/Release-iphoneos/*