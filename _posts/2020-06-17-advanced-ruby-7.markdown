---
layout:     post
title:      "Advanced Ruby: duck typing"
date:       2020-06-17 04:50:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   advanced-ruby-7
---

# Advance Ruby: duck typing

Ruby is a dynamic typing language. This means that declaring a variable doesn't fix the type of it. The class of every variable in Ruby depends on the class of the object assigned to it. This class can be changed.

In Ruby the way to interact with objects isn't fixed by its class. The way to interact is choose by the developer. if an object need to respond to a method and it responds, then it can be called, it doesn't matter its class.

Duck typing is not a intrinsic mechanism of the Ruby language. It is a convention that developers choose to adopt. 

```ruby
'Hello World!'.respond_to?(:to str)  # => true  
42.respond_to?(:to_str)  # => false  
```

Remember that in Ruby there is no implicit type-checking.

This type of design convention leads to adopt another type of convention: the method name convention. Most of the Ruby methods are sort and very reusable names. This tendency help developers to design its classes in a reusable way.

