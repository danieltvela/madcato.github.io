---
layout:     post
title:      "Advanced Ruby: metaclasses"
date:       2020-05-20 13:38:00
author:     "Daniel Vela"
og_image: "/img/post-bg-19.jpg"
locale:       en
lang-ref:   advanced-ruby-5
---

# Advance Ruby: metaclasses

Each object in Ruby has a hidden class. Even classes in Ruby, that are also objects, have a hidden class associated.

```ruby
class Person
  def name(n)
    @name = n
  end
end

thisjs = Person.new  # => :name
thisjs.class  # => Person
thisjs.singleton_class  # => #<Class:#<Person:0x00007f9c8fb302a8>>
```

You can add methods to the metaclass of an instance with:

```ruby
juan = Person.new
juan.name("Juan")

def juan.last_name 
  "Perez"
end

juan.last_name  # => "Perez"

tom = Person.new
tom.last_name  # => NoMethodError (undefined method `last_name' for #<Person:0x00007f9c8f085650>)
```

Adding a method to the metaclass of an object doesn't add that method to the class of that object.