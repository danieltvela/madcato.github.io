---
layout:     post
title:      "Using static libraries in Xcode 4"
date:       2012-01-29 16:32:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   static-libraries-xcode4
---


1. First create the library project.
2. For each header file make sure it is set as public in the **“Target Membership”** section of the inspector pane.
	![inspector pane]({{ site.url }}/assets/tumblr_inline_mk0bnbJBhO1qz4rgp.png)
3. Set the following settings in build settings.
	* Installation directory => **$(BUILT_PRODUCTS_DIR)**
	* Skip Install => **Yes**
	* Public Headers Folder Path => **$(TARGET_NAME)**
	![inspector pane]({{ site.url }}/assets/tumblr_inline_mk0bnuQ9oj1qz4rgp.png)
4. Now create a new workspace for your project.
5. Add the static library to your project.
6. Add build target dependencies. Select apps build target and add the static library to the **“Link binary with libraries”** build phase.
	![inspector pane]({{ site.url }}/assets/tumblr_inline_mk0bocxZpq1qz4rgp.png)
7. Add the static libraries headers. Open **“Build Settings”** tab and locate **“User Header Search Paths”**. Set to **$(BUILT_PRODUCTS_DIR)**
	![inspector pane]({{ site.url }}/assets/tumblr_inline_mk0bolVe9C1qz4rgp.png)
8. Configure the projects scheme. Edit your project scheme and add the static library build target before our apps build target.
9. Last include in the project precompiled header the library main headers (This step is optional)


20110718.7