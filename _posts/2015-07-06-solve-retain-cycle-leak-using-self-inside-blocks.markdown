---
layout:     post
title:      "To solve retain cycle leak using self inside blocks"
date:       2015-07-06 01:50:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   solve-retain-cycle-leak-using-self-inside-blocks
---

{% highlight objc %}
__weak id weakSelf = self;
[someObject someMethodWithBlock:^{    
[weakSelf someOtherMethod];
}] 
{% endhighlight %}