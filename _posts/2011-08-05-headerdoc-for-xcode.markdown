---
layout:     post
title:      "HeaderDoc for Xcode"
date:       2011-08-05 09:24:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   headerdoc-for-xcode
---

For more information, follow this link: [HeaderDoc](http://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/HeaderDoc/intro/intro.html)

## scripts

Run:

{% highlight bash %}
$ headerdoc2html MyHeader.h
{% endhighlight %}

This create a directory named MyHeader_H with an html documentation.

Run:
{% highlight bash %}
$ gatherheaderdoc MyHeader_H
{% endhighlight %}

For scanning the input directory for doc html and it creates and index for that doc.

## Function doc sample

{% highlight objc %}
/*!  
    @function FunctionName  
    @param k param doc.  
    @return a string.  
    This is a comment about FunctionName  
*/  
char* FunctionName(int k);  
{% endhighlight %}

## Class doc sample

{% highlight bash %}
/*!  
    @class IOCommandGate  
    @abstract This is a introductory explanation about the class.  
    @discussion This is a more detailed description of the class.  
    @throws foo_exception  
    @throws bar_exception  
    @namespace I/OKit  
    @updated 2003-03-15  
*/  
class IOCommandGate : public IOEcentSource  
{  
...  
}  
{% endhighlight %}

## More doc tags

@availabilitymacro, @class, @category, @protocol, @const, @define, @function, @enum, @title, @framework, @functiongroup, @header, @method, @property, @struct, @typedef, @union, @var

## Other @class doc subtags

@classdesign, @coclass, @depency, @helper, @helps, @instancesize, @ownership, @performance, @security, @superclass

## Other @function and @method doc subtags

@param, @result, @templatefield, @throws

20110324.50

