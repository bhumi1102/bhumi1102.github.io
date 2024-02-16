---
layout: post
title:  "Dynamically updating method definitions in Ruby: `send`, `define_method` and more!"
---

This one is all about methods in ruby. How we can dynamically call, define and remove methods. The difference between module methods, class methods, instance methods and singleton methods. We also cover how methods are also just objects and an interesting example of updating a method definition at runtime.

**Dynamically Calling Methods**

In ruby we can call any method on any object. In formal OO programing computer science terminology, *calling a method* is thought of as *sending a message*, where the object is the *receiver*. Ruby has a method named `send` on the `Object` class. So all ruby objects inherit `send` and here's how we 'call methods' using `send`. 

```ruby
?> class Hello
?>   def say_hello(name)
?>     "Hello, #{name}"
?>   end
>> end

>> h = Hello.new
=> #<Hello:0x0000000107b127b0>
>> h.say_hello("Ruby")
=> "Hello, Ruby"

>> h.send(:say_hello, "Ruby")
=> "Hello, Ruby"
```

The first argument to `send` is the method name (as a symbol or a string) or the _message_ that you are trying to send. The remaining arguments (including a block) are passed on to the method as parameters. 

So `send` allows us to call methods dynamically by sending a message to the object as the receiver. Why would be want to use `send` instead of the regular dot `obj.my_method` syntax? 

Well, with `send` the method name can be decided at runtime. The method name is a regular parameter and we can decide which method to call at the very last minute, dynamically. This is called Dynamic Dispatch.

The other reason is that with `send`, we call any method, including *private* methods. For example `method_missing` is actually a private method of `BasicObject` but we call it with `send` like this `obj.send(:method_missing, :random_method)`.

We can, not only *call* methods dynamically, but also *define* methods dynamically. Let's see how that works.

**Dynamically defining methods**

`define_method` is a method of the `Module` class. It takes a block which becomes the body of the new method being defined.

```ruby
?> class MyClass
?>   define_method :ask_me_anything do |question|
?>     "The answer to your question, #{question}, is 42"
?>   end
>> end

>> obj = MyClass.new
>> obj.ask_me_anything("what is the meaning of life, the universe, and everything")
=> "The answer to your question, what is the meaning of life, the universe, and everything, is 42"
```

`ask_me_anything` becomes an instance method of `MyClass`. The reason to use `define_method` instead of `def` keyword is again the fact that we can wait to decide the name of the method at runtime. This is powerful and allows for much of the functionality in Rails, for example.

We can also undefine or remove methods like this:

```ruby
?> class Hello
?>   def greet
?>     "Hello"
?>   end
>> end

>> h = Hello.new
>> h.greet
=> "Hello"

>> Hello.remove_method(:greet)
>> h.greet
(irb):9:in `<main>': undefined method `greet' for #<Hello:0x00000001032316e0> (NoMethodError)
```

Both `undef_method` and `remove_method` are methods on the `Module` class. `undef_method` is more drastic in that it will undefine any method, including inherited ones. `remove_method` will remove the method from the receiver object only.

There is one more thing I've wondered about, in terms of **terminology around method types**. We say modules have class methods and instance methods. What is meant my module class methods. And also, modules cannot be instantiated (that's the main difference between modules and classes) so how do think of module instance methods? Let's clear this up.

**class vs. module vs. instance vs. singleton methods**

What are module methods? Let's see in `IRB`:

```ruby
?> module Hello
?>   def self.greet
?>     "Hello there!"
?>   end
>> end

>> Hello.greet
=> "Hello there!"
```

**Module methods** are the ones defined with `self.` inside a module definition. They are called with the module name like `Hello.greet` in the example above.

Same deal with **class methods**. They are the ones defined with `self.` inside a class definition. I do see 'class methods' and 'module methods' used interchangeably when referring to methods like `Hello.greet` inside modules. And that's accurate in a way.

So what about the term 'module instance methods'? 

```ruby
?> module MyModule
?>   def instance_method
?>     "a class that includes this module will get this method"
?>   end
>> end

?> class MyClass
?>   include MyModule
>> end

>> obj = MyClass.new
>> obj.instance_method
=> "a class that includes this module will get this method"
```

When a module is included in a class, the **module's instance methods** (the ones *without* `self.`) become instance methods of that class. That is all that's meant by 'module instance methods'. They are the ones that a class gains when it mixes in a module with `include`. ([this post](https://theleafnode.com/include-vs-extend-whats-the-difference/) talks more about include, extend, etc.)

That clears up this comment in the [Kernel docs](https://ruby-doc.org/core-3.0.2/Kernel.html) for example.

>Kernel module has some module methods and the rest are instance methods documented in `Object` class, which includes `Kernel`

Before we wrap up, another type of method is singleton methods. 

Did you know that, in ruby, objects can have their own methods? A method that 'belongs to' an object. No other object, even of the same class, has that method. These are called **singleton methods** ([this post](https://theleafnode.com/what-is-a-singleton-class-in-ruby/) of One Ruby Question is all about singletons methods and singleton classes. So I won't repeat here).

I will leave you with one last fact about methods.

All methods in Ruby are objects of the class `Method`. So methods are also just objects in Ruby, like everything else. We can do interesting things like **updating a method definition** like this:

```ruby
?> class MyClass
?>   def my_method
?>     puts "original method"
?>   end
>> end

>> obj = MyClass.new
>> obj.my_method
original method
                                          
>> original_method = obj.method(:my_method)  # save a copy of old method
=> #<Method: MyClass#my_method() (irb):2>
?> obj.define_singleton_method(:my_method) do
?>   puts "before calling old method"
?>   original_method.call
?>   puts "after calling old method"
>> end

>> obj.my_method  # we have updated the method definition
before calling old method
original method                                                   
after calling old method 
```

There is a lot going on here. 
- We are *updating* the definition of `my_method` dynamically to add functionality to it (as a singleton method on the object `obj`). 
- We are saving a copy of the original method and calling it inside the updated method definition.
- We are not changing the contract `my_method` has with its callers. We are not changing the methods 'api' to the outside world. But the existing callers will get the added functionality for free, without doing anything different or even knowing about it. 

You can imagine the use cases that this pattern would enable. This is the power of ruby (that it lends to rails).


P.S. In addition to methods, we also have `lambdas` (and `procs`) as a way of storing pieces of reusable code. While procs behave like blocks, lambdas behave like methods.  We can think of lambdas as anonymous methods created with `lambda` or `->` syntax. (`lambda` and `proc` is one of the more confusing features of Ruby. I will plan to make a future post on this topic).