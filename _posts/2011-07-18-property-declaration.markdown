---
layout:     post
title:      "@property attributes"
date:       2011-07-18 14:43:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   2011-07-18-property-declaration
---

![xcode icon]({{ site.url }}/assets/tumblr_inline_mjuuzjzbF01qz4rgp.jpg)

## Property declaration:

{% highlight objc %}
@property(attributes) type name;
{% endhighlight %}

## Accessor method names

* **getter=getterName** Specifies the name of the getter method.
* **setter=setterName** Specifies the name of the setter method.

## Writability

* **readwrite** Read and write property.
* **readonly** Property can be only red.

## Setter semantics

* **assign** The setter is simple assignment. You typically use this setter with non-pointer variables like: NSInteger, float and CGRect.
* **retain** Specifies that retain will be called each time a new assignment is done. The previous value of the property will be released.
* **copy** A copy of the object will be used in the assignment. No retain is called, but a release is send to the previous object.

## Atomicity

* **nonatomic** It’s used in multithreaded environments to avoid that two or more different thread access the same property at same time.
