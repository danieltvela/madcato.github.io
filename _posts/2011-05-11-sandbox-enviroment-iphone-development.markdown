---
layout:     post
title:      "Sandbox environment for iPhone development"
date:       2011-05-11 12:22:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   sandbox-enviroment-iphone-development
---

Today I suffered from a bizarre failure of Xcode that make us lose time. When attempting to access one of my in-app Purchases, the framework StoreKit returned my ID on invalidProductIdentifiers property as if the ID not exist.

The strange thing is that yesterday worked. I had no problem when I created the ID, but today failed. What was different today?

After much searching online I found the solution. For Sandbox environment to work correctly, the app must be installed from the Xcode debugging process. This means you have to delete any earlier versions you have installed from the App Store, because these applications do not fall within the Sandbox environment and therefore have no access to in-app Purchases test environment of iTunes Connect.

This same error can cause other failures related to the test environment, such as iAds.