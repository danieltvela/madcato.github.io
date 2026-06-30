---
layout:     post
title:      "How to marshal a Ruby object"
date:       2015-03-22 15:41:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   marshal-ruby-object
---


To serialize and to restore objects in Ruby use **Marshal** class.

[Marshal class documentation](http://ruby-doc.org/core-1.9.3/Marshal.html)

{% highlight ruby %}
class Klass
  def initialize(str)
    @str = str
  end
  def say_hello
    @str
  end
end


o = Klass.new("hello\n")
data = Marshal.dump(o)
obj = Marshal.load(data)
obj.say_hello  #=> "hello\n"
{% endhighlight %}