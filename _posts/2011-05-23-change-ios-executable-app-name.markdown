---
layout:     post
title:      "Change iOS executable app name"
date:       2011-05-23 13:19:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   change-ios-executable-app-name
---

The correct way to change the executable name of an iOS app is to change the **Product name** in the target settings.

It must be in the target settings. if you change the executable name in the project plist file you can get code signing errors like: *"object file format invalid or unsuitable"*