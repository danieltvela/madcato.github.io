---
layout:     post
title:      "Advanced Ruby: Object-Specific Classes"
date:       2020-05-02 00:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-16.jpg"
locale:       en
lang-ref:   advanced-ruby-2
---

# Advance Ruby: Object-Specific Classes

In Ruby you can create a class tied to an specific object. With this mechanism you can include methods to an existing object.

Sample: 

```ruby
a = "Hello World!"

class << a
  def to_s
    "The value is '#{self}'"
  end
end

a.to_s  # "The values is 'Hellow World!'"
```

The notation *class << obj* allows us to include new methods to the object *obj*.

