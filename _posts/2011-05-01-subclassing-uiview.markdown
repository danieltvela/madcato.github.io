---
layout:     post
title:      "Subclassing UIView"
date:       2011-05-01 11:26:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   subclassing-uiview
---

When subclassing an UIView and you need to create a method for initialization, then you must choose at least one of the following.

* **initWithFrame**. This is the default initializator. Implement it when your class is created from code.
* **initWithCoder**. Implement this method when your UIView subclass will be created by the Interface Builder.
* **layerClass**. Implement this when the view will be draw by an alternative method like OpenGL ES.

20110324.30