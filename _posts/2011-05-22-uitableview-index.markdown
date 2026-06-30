---
layout:     post
title:      "UITableView index"
date:       2011-05-22 13:19:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   uitableview-index
---

You can generate the index of the tables with the class **UIlocalizedIndexedCollation (UIKit)**.

Generates an index from **A** to **Z**, including the **#** symbol.

To add the symbol search, add the predefined constant **UITableViewIndexSearch** to the array.

Finally, to move the table in the position of the browser, use the following code in the function **sectionForSectionIndexTitle**.

{% highlight objc %}
if([title isEqualToString:UITableViewIndexSearch]){  
    [self.tableView setContentOffset:CGPointZero animated:NO];  
}  
{% endhighlight %}

![table index]({{ site.url }}/assets/tumblr_inline_mjum47E6dg1qz4rgp.png)