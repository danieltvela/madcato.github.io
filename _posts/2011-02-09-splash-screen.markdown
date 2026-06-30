---
layout:     post
title:      "iPhone - Splash screen"
date:       2011-02-09 16:03:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   splash-screen
---

Declaration(.h):

{% highlight objc %}
UIImageView *splashView;  
{% endhighlight %}

Implementation(.m):

{% highlight objc %}
splashView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 320, 480)];  
splashView.image = [UIImage imageNamed:@"Default.png"];  
[window addSubview:splashView];  
[window bringSubviewToFront:splashView];  
// Do your time consuming setup  
[NSThread sleepForTimeInterval:2.1];  
[splashView removeFromSuperview];
{% endhighlight %}