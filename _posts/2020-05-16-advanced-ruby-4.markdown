---
layout:     post
title:      "Advanced Ruby: class << self"
date:       2020-05-16 15:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-18.jpg"
locale:       en
lang-ref:   advanced-ruby-4
---

# Advance Ruby: class << self

With `class << self` every method declared after this becomes a class method.

```ruby
class Person
  class << self
    def once(*ids)
    # code
    end
  end
end
```

But if `class << self` is declared inside an instance method, then `self` relates to the object being called. This means that the following declarations affect that object and not its class.

```ruby
class Person
  def process
    method2
  end
private
  def process1
    class << self
      alias process process2
    end
  end
  def process2
    class << self
      alias process process1
    end
  end
  
  alias process process1
end
```

This code makes that every time method `process` of an instance of class `Person` is called, an `alias` provokes that the next time `process` is called, the function called will be different: from `process1` will change to `process2` and so on.