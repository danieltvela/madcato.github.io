---
layout:     post
title:      "BUG: Invalid content size ‘ADBannerContentSizeLandscape’"
date:       2011-03-09 10:07:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   invalid-content-size-ADBannerContentSizeLandscape
---

**reason: ‘Invalid content size 'ADBannerContentSizeLandscape’ passed to ADAdSizeForBannerContentSize’**

The problem comes from version incompatibilities. The identifier **ADBannerContentSizeLandscape** works only for version 4.2. If you have declared the **AdBannerView** inside a xib, this identifier will crash your aplication when running in a 4.0 or 4.1 iOS iPhone.

To solve this problem follow the steps described in: [http://www.raywenderlich.com/1371/how-to-integrate-iad-into-your-iphone-app](http://www.raywenderlich.com/1371/how-to-integrate-iad-into-your-iphone-app)

