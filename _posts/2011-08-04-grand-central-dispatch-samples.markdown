---
layout:     post
title:      "Grand Central Dispatch samples"
date:       2011-08-04 19:01:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   grand-central-dispatch-samples
---

You can use GCD for thread substitutions. It needs blocks.

Example of use:

{% highlight objc %}
 - (void)doTimeConsumingOperation:(id)operation {  
    dispatch_queue_t queue;  
    queue = dispatch_queue_create("com.example.operation",NULL);  
    dispatch_async(queue,^{  
       [operation doOperation]; // Main op.  
    });  
    dispatch_release(queue)  
 }  
{% endhighlight %}
You can use **dispatch\_sync** instead **dispatch\_async** for locked code execution.

Another sample of code is this way to perform a delayed selector on an object:

{% highlight objc %}
 - (void)doLaterOperation:(id)operation {  
      dispatch_time_t delay;  
      delay = dispatch_time(DISPATCH_TIME_NOW, 50000 /* nanoseconds */);  
      dispatch_after(delay,^{  
           [operation doOperation]; // Main op.  
      });  
      dispatch_release(delay);
 }  
{% endhighlight %}

Finally other fine use of GCD is for creating safe singleton objects:

{% highlight objc %}
 +(MyClass *) sharedInstance {  
      static dispatch_once_t predicate;  
      static MyClass *shared = nil;  
      dispatch_once(&predicate, ^{  
          shared = [[MyClass alloc] init];  
      });  
      return shared;  
 }  
{% endhighlight %}

With **dispatch\_once** the code only will be executed one and only one time.

Available in iOS 4.0 and later.    
Available in Mac OS X v10.6 and later.    
20110324.26