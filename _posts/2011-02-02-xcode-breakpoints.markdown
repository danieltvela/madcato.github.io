---
layout:     post
title:      "Xcode breakpoints disabled in Release mode"
date:       2011-02-02 15:19:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   xcode-breakpoints
---

In the last version of xcode (3.2.5) breakpoints are disabled when debugging the Release version.

To solve this problem go to the Preferences, then Debugging, and uncheck "Load symbols lazily". This fix the issue.  

![Xcode preferences]({{ site.url }}/assets/Xcode_breakpoints_disabled_in_Release_mode.png)