---
layout:     post
title:      "Advanced Ruby: Mixins"
date:       2020-04-26 00:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-15.jpg"
locale:       en
lang-ref:   advanced-ruby-1
---

# Advance Ruby: Mixins

Have you ever wish to add some methods or properties to an existing class, without changing it? With mixins you can.

A mixin is a module where come methods are declared. This mixin can be 'included' into a class. This way, that class would has the mehtods declared in the mixin. This removes the need of multiple inheritance in Ruby. 

Sample: 

```ruby
module Loggable  # A mixin must be declared as a module
  def toLog
    print("Object: #{self.id} of class: #{self.type.name}")
  end
end

class Person
  include Loggable  # This add the method 'toLog;' to this class
  
  def initialize(name)
    @name = name
  end
end
```

Can a 'mixin' add some instance variable to another class? Yes, the same way Ruby does: the first time a @-variable is declared, the property is created.

```ruby
module ErrorDescriptable
  attr :errorDescription
  def addError(str)
    @errorDescription = "Error: #{str}"
  end
end

class Person
  include  ErrorDescriptable
  def initialize(name)
    if name.nil?
      addError("Invalid name")
    end
    @name = name
  end
end
```

This is a powerful way to extend a class. But also modules can be used to extend existing object. Yes, already instantiated object. Like:

```ruby
module ErrorDescriptable
  attr :errorDescription
  def addError(str)
    @errorDescription = "Error: #{str}"
  end
end

a = "Hello world"
a.extend ErrorDescriptable  # Add the module to an object
a.addError("Test")
```

Also this trick can be used to add method to a class, as class methods

```ruby
module ErrorDescriptable
  attr :errorDescription
  def addError(str)
    @errorDescription = "Error: #{str}"
  end
end

class Person
  include  ErrorDescriptable  # isntance method
  extend ErrorDescriptable  # class method
end

Person.addError("Test1")
p = Person.new
p.addError("Test2")
```



