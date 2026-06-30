---
layout:     post
title:      "Advanced Ruby: module\_eval"
date:       2020-05-13 14:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-17.jpg"
locale:       en
lang-ref:   advanced-ruby-3
---

# Advance Ruby: module\_eval

With the expresion `module_eval` passing one string like argument, you can define methods dinamically. Yes, you read it right: you can create methods to a class, module or object, to any context dinamically. Write a string with code that generates some method, and with `module_eval` you can inject that code into the current context.

This is a sample writen by Tadayoshi Funaba:

```ruby
def ExampleDate.once(*ids)
  for id in ids    
    module_eval <<-"end_eval".   
      alias_method :__#{id.to_i}__, #{id.inspect}    
      def #{id.id2name}(*args, &block)    
        def self.#{id.id2name}(*args, &block)    
          @__#{id.to_i}__    
        end    
        @__#{id.to_i}__ = __#{id.to_i}__(*args, &block)    
      end    
    end_eval
  end
end
```

This code is going to be executed once, thanks to the `once` expression behind the method name. Then for each element in the parameters, the `module_eval` is going to create an alias of a method and define a new method.

