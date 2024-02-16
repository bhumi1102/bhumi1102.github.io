---
layout: post
title:  "What is the value of 'self' in different contexts?"
---

Ruby doesn't have very many keywords. Almost everything is a method call. But there is one important keyword, `self`, that is useful to understand.

In this post, we'll see the value of `self` in different contexts - inside a method call, inside a class or module definition, and even at the top-level in an `irb` session.

We will also see how to use methods like `instance_eval` and `class_eval` to change the value of `self`.

Let's start with some code:

```ruby
$ irb    # starting a brand new irb session
>> self
=> main
>> self.class
=> Object
```

Everything in Ruby is an object. And all code is evaluated in the context of an object. Ruby keeps track of the 'current object' with the keyword `self`. That is what `self` is, it holds the 'current object'. 

As we can see in the `irb` session above, **the value of `self` at the top-level** is `main`. `main` is just an object, it's an object of class `Object`. And it's the 'current object' at the top-level.

**The value of `self` inside a class (or module) definition** is the class itself, which is an object of the class `Class`. (This is getting repetitive but I'll say it again :) Everything in Ruby is an object. Classes are just objects.).

```ruby
?> class Hello
?>   puts "value of self is #{self}"
?>   class << self
?>     puts "value of self inside singleton class is #{self}"
?>   end
>> end
value of self is Hello
value of self inside singleton class is #<Class:Hello>
```

In the example above, we can see the value of `self` inside a class definition. We also see the value of `self` inside the singleton class of the class we're defining. (If you read the post about singleton classes, this should be familiar).

**The value of `self` inside method calls** is the object that is the receiver of the method call. For example:

```ruby
?> class MyClass
?>   def my_method
?>     puts "value of self is #{self}"
?>   end
>> end

>> foo = MyClass.new
=> #<MyClass:0x00000001112dd590>
>> foo.my_method
value of self is #<MyClass:0x00000001112dd590>
```

Now, what if we need to change the value of `self` in our program. We can use `instance_eval` or `class_eval`. They both change the value of `self` but they are different. Let's see examples of how to use both.

**instance_eval**
`instance_eval` is an instance method of `BasicObject`. It takes a block and evaluates the code in the block in the context of an object. 

In the example below, we can see that the block passed to `instance_eval` can access the receiver object's private methods and instance variables. 

```ruby
?> class Hello
?>   def initialize
?>     @my_var = "hello"
?>   end
>> end

>> obj = Hello.new
=> #<Hello:0x000000010fd677b0 @my_var="hello">
?> obj.instance_eval do
?>   puts self
?>   puts @my_var
>> end
#<Hello:0x000000010fd677b0>
hello       
```

We can access the instance variable `@my_var` on the receiver object `obj`. In addition, notice that the value of `self` inside the block is the object `obj`. And that's because we are inside a method call, `instance_eval` is a method. So the value of `self` is consistent with what we learned above. 

If we change the value of an instance variable inside an `instance_eval` block, that *will* change the value for the object as well. 

```ruby
>> obj.my_var
=> "hello"

?> obj.instance_eval do
?>   @my_var = "bye"
>> end

>> obj.my_var
=> "bye"
```

**class_eval**
`class_eval` is an instance method of the `Module` class. It evaluates a block in the context of an existing class.

In addition to the 'current object', Ruby also keeps track of the 'current class'. But that is a bit hidden, there is no keyword like `self` to get at the current class.

`class_eval` changes the value of `self` *and* the 'current class'. It effectively reopens the class, similar to using the `class` keyword.

```ruby
?> def add_a_method_to_class(c)
?>   puts "outside: #{self}"
?>   c.class_eval do
?>     def some_method
?>       puts "inside: #{self}"
?>       "hello"
?>     end
?>   end
>> end

>> add_a_method_to_class String  # will open String
outside: main
                                         
>> "ruby".some_method
inside: ruby
=> "hello"        
```

However, `class_eval` is more flexible than the `class` keyword. For example, `class` keyword requires that we have a constant. But we can use `class_eval` with any variable that references a class, such as `c` in the example above.

And `class` keyword opens a new scope, loosing sight of the outer bindings. On the other hand, we can reference variables from the outer scope in a `class_eval` block.

To wrap up, here are some facts to take away:

- Ruby keeps track of the 'current object' with the keyword `self`. That is what `self` is, it holds the 'current object'.
- Ruby also keeps track of the 'current class'. All methods defined with `def` keyword became instance methods of the current class.
- `BasicObject#instance_eval` and `Module#class_eval` are methods that take a block. They both change the value of `self`. `class_eval` also changes to value of the 'current class'.

Side note: there is also `instance_exec` and `class_exec` and they are the same as the `*_eval` methods, except that they allow you to pass arguments to the block.

Here's one more way to think about when the value of `self` will change. You may have heard the term *scope gate*. When we use the keywords `def`, `class`, or `module` to define methods, classes, and modules in our code, we are opening up a new *scope gate*. The value of `self` changes as we cross these *scope gates*.

That's all I got for this one. 