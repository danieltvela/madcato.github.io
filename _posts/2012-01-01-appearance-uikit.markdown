---
layout:     post
title:      "Appearance of UIKit"
date:       2012-01-01 11:09:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   appearance-uikit
---


\* iOS 5 required

With appearance kit we can set the default attributes of any UIKit control. This attributes include tint color or background.

## Changing background

NavBar

{% highlight objc %}
setBackgroundImage: forBarMetrics:  
    typedef enum{  
         UIBarMetricsDefault,  
         UIBarMetricsLandscapePhone,  
    } UIBarMetrics;  
{% endhighlight %}

UIBarButtons(back button)

{% highlight objc %}
setBackButtonBackgroundImage: forState: barMetrics:  
{% endhighlight %}

TabBar

{% highlight objc %}
UIImage* tabBarBackground = [UIImage imageNamed:"tabbar"];  
[[UITabBar appearance] setBackgroundImage:tabBarBackground];  
[[UITabBar appearance] setSelectionIndicatorImage:[UIImage imageNamed:"selection"]];  


[<UIControl> appearance] "setAttributte"
{% endhighlight %}

With this invocation we can set the default attributes of any UIKit controls.

![ishop]({{ site.url }}/assets/tumblr_inline_mk0ba3zL1u1qz4rgp.png)
