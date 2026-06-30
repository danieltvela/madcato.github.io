---
layout:     post
title:      "Ignoring build directory for svn"
date:       2011-03-02 23:52:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   ignoring-build-directory-svn
---

The build directory in every Xcode project can be ignored using this svn command:

{% highlight bash %}
svn propset svn:ignore . build  
{% endhighlight %}

When you make a **svn stat** you will see this files marked as ?. But i fyou try to import or add it to the project, it will be ignored.

[SVN Documentation](http://svnbook.red-bean.com/en/1.1/ch07s02.html)