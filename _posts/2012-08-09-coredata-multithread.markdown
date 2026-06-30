---
layout:     post
title:      "CoreData with multithread"
date:       2012-08-09 12:14:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   coredata-multithread
---


If we use CoreData in a multithreaded applications, we must take count of the following. A **NSManagedObject** created in one thread, can’t be used in another thread. Also a **NSManagedObjectContext** can’t be used inter threads.

Two practices are recommended when programming with threads and CoreData:

1. Create a new **NSManagedObjectContext** for each thread. The object **NSPersistentStore** can be shared among thread safely. 
2. When is necessary sharing a **NSManagedObject** between threads, share its objectID property only. Then query that object with method:
		{% highlight objc %}
		[managedObjectContext objectWithID:objectID];
		{% endhighlight %}

As apple say:

![apple doc]({{ site.url }}/assets/tumblr_inline_mk0cy6QSYr1qz4rgp.png)

PS: Remember that when using asynchronous request -either using blocks or delegates-, always is used a different thread for the response treatment.