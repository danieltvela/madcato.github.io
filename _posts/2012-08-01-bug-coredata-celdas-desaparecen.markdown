---
layout:     post
title:      "Bug CoreData - cells dessapear"
date:       2012-08-01 18:55:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   bug-coredata-celdas-desaparecen
---

Many users of LiShop had camplained about the list dessapearing when creating many articles.

Reviewing the logs I discover the next error:

**Aug 1 18:06:38 unknown ListaCompra2[19674] \: CoreData: error: (NSFetchedResultsController) The fetched object at index 6 has an out of order section name ‘Appetizers. Objects must be sorted by section name’**

The problem was provoked because when objects are sorted by a property, this ordering must correspond with the sort of sections of the **NSFetchRequest**.

To solve put the folloing parameter in the constructor of **NSSortDescriptor** of text fields. To avoid sort problems with uppercase and lowercase.

{% highlight objc %}
selector:@selector(caseInsensitiveCompare:)
{% endhighlight %}

It must be:

{% highlight objc %}
NSSortDescriptor *sortDescriptor1 = [[NSSortDescriptor alloc] initWithKey:@"name" ascending:YES selector:@selector(caseInsensitiveCompare:)];
{% endhighlight %}