---
layout: post
title:  "What is a singleton class in Ruby?"
---

What are singleton classes in Ruby? Why do we need them? Or rather what (language design) problem do they solve? 

Side note: singleton classes are also known as meta classes or eigen classes. In case you've heard those terms.

To get at the motivation for singleton classes in Ruby, first we need to learn about singleton methods. 

All objects in ruby can have their own methods. That 'belong to' the object and *not* to the object's class (the methods in the object's class are called regular instance methods). These are called **singleton methods** of that specific object. 

Let's see some examples of defining singleton methods on an plain object and on a string:

```ruby
>> obj = Object.new
=> #<Object:0x000000010c3216c8>
?> def obj.a_method
?>   "hi"
>> end

>> obj.singleton_methods
=> [:a_method]

>> my_string = "a plain old string"
?> def my_string.caps?
?>   self.upcase == self
>> end

>> my_string.caps?
=> false
>> my_string.singleton_methods
=> [:caps?]

>> different_string = "a different string"
>> different_string.caps?
(irb):70:in `<main>': undefined method `caps?' for "a different string":String (NoMethodError)
```

Singleton methods 'belong to' the object `obj`. And no other object, even of the same class as `obj.class`, has these methods. 

Okay cool. Object's can have their own methods. The next question is how does the ruby interpreter find them when we call these singleton methods `obj.my_singleton_method`?

**Method Lookup and Singleton Classes**

We know that method lookup is the process used to find method definitions in Ruby. We start with the object's class and go up the ancestor chain (all the way to `method_missing` as we saw in a previous post). It looks something like this: 

 ![Untitled-2022-10-07-1312(2).png](https://assets.buttondown.email/images/91113480-20ea-4579-ad1f-153d3d53c20b.png?w=960&fit=max) 

`say_hello` is an instance method of the class `Hello`. How does Ruby find `my_singleton_method` of the object `obj?` Which box does that go in?

We need another box in the picture where we can put an object's singleton methods. This is where singleton classes come in. The box labeled `#obj` is called the object's **singleton class**. All objects in Ruby have a singleton class.

 ![Untitled-2022-10-07-1312(3).png](https://assets.buttondown.email/images/ec29425b-18ee-44ff-bd0b-4a265da4bd2f.png?w=960&fit=max) 

So, to answer our original question: a singleton class is a place to keep an object's singleton methods.

Side note: the syntax `#obj` is short of "the object `obj`'s singleton class". I don't think it's some official syntax, I saw it in the metaprogramming ruby so I'm using it.

Coming back to method lookup, when we call a method on an object `obj`, Ruby will first look in the object's singleton class `#obj` and *then* go up the ancestor chain. That is how we find `my_singleton_method` on `obj`. 

Singleton classes are part of the ancestor chain of an object. But methods like `Object.class` and `Module.ancestors` keep them carefully hidden. They do not show up in the ancestors array as you can in the example below.

```ruby
>> obj = Hello.new
=> #<Hello:0x000000010c208a48>
?> def obj.my_singleton_method
?>   "something"
>> end

>> Hello.ancestors
=> [Hello, MyModule, Object, PP::ObjectMixin, Kernel, BasicObject]

>> obj.singleton_class
=> #<Class:#<Hello:0x000000010c208a48>>
>> obj.singleton_class.superclass
=> Hello
>> obj.class
=> Hello
>> obj.singleton_class.superclass == obj.class
=> true
```

From the example above we make this interesting observation though. That **the superclass of `obj`'s singleton class is the object's class**. That sounds confusing to read but if we look at the picture above and the `irb` session, it becomes very clear. This proves that the singleton class *is* in the ancestor chain for method lookup.

One more thing, Ruby didn't always have the `sigleton_class` method. In earlier versions of Ruby, in order to get a reference to the singleton class of an object, we'd have to open it up and return `self` from the right scope. Like this:

```ruby
>> obj = Object.new
=> #<Object:0x000000010bb24b50>
?> singleton_class = class << obj
?>   self
>> end
=> #<Class:#<Object:0x000000010bb24b50>>
```

We can see from the object ids that `singleton_class` is related to the original `obj`. But these days we can get a reference to any object's singleton class with the handy `Object#singleton_class` method.

Some of this may sound complex but there is elegance in the consistency of this design. Before we wrap up, let's talk about class methods to see what I mean.

**Class Methods as Singleton Methods**

Recall that all classes in Ruby are just objects. They are instances of the class `Class`. 

```ruby
>> class Foo; end

>> Foo.class
=> Class
>> Foo.superclass
=> Object
```

The syntax for calling a method on an object `obj.a_method` is the same as calling a class method on a class `Foo.a_class_method`. 

There are also parallels between how we define a class method on a class and how we define a singleton method on an object.

```ruby
# syntax for defining singleton methods on an object
>> obj = Object.new
?> def obj.my_singleton_method
?>   "yo"
>> end

>> obj.my_singleton_method
=> "yo"

# typical syntax for class method definition

?> class Foo
?>   def self.my_class_method
?>     "hello"
?>   end
>> end

>> Foo.my_class_method
=> "hello"

# class method definition that look more like singleton method
?> def Foo.another_class_method
?>   "bye"
>> end

>> Foo.another_class_method
=> "bye"
```

So we have all been using singleton methods all along. **Class methods are a special case of singleton methods.** They are methods on the object of the class `Class` that we are defining. And therefore class methods 'live in' that class's singleton class.

That all comes together nicely. Now you know about these hidden singleton classes in Ruby.


P.S. now with the mystery of singleton methods and classes illuminated, you could reread the previous post about the difference between `exclude` and `include`. Some of the hand wavy statements about how "singleton classes 'hold' class methods" should be clearer. There is a bit more to say about Singleton classes and inheritance, I'll save that for another post as this one is getting longer than usual.

P.P.S I should make one last point here: we're talking about singleton _objects_, not the Singleton _Pattern_. The [Singleton Pattern](https://en.wikipedia.org/wiki/Singleton_pattern) is design pattern that I have seen in other languages like C++ and Java to restrict class instantiation to one object. The term 'singleton' in ruby was confusing to me at first so thought I'd mention that singleton objects in ruby don't have anything to do with the singleton pattern.