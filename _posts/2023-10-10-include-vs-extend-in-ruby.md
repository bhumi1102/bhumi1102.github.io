layout: post
title:  "Include vs. Extend, what's the difference?"
---

What is the difference between `include` vs. `extend` in Ruby? And how do we know which one to use?

Note: the explanation below mentions *singleton classes*. For now think of a singleton class (aka metaclass or eigenclass) as a place where class methods are 'held'. As opposed to instance methods which are 'held' in the class itself. All classes in Ruby have a singleton class. Not very clear, I know. Nor is a complete story. So I will make singleton classes a topic of a future post.

Let's start with the motivation for `include` and `extend`.

Assuming we have a module `Hello` with a 'class method' `say_hello`:

```ruby
?> module Hello
?>   def self.say_hello
?>     "hello"
?>   end
>> end
```

We want to add this module's functionality in our class `Greeting`. Specifically we want the class method `say_hello`. So we can just `include` the module and that should do the trick right?

```ruby
?> class Greeting
?>   include Hello
>> end

>> Greeting.say_hello
(irb):28:in `<main>': undefined method `say_hello' for Greeting:Class (NoMethodError)
```

No. That's doesn't work. And here's why.

When a class includes a module, it gets the module's *instance* methods, **not** the class methods. The class methods stay tucked away, in the module's *singleton class*.

What we have to do to add `say_hello` as a class method to our target class `Greeting` is to include the module `Hello` in the class's singleton class. To do this, we have to open `Greeting's` singleton class like this:

```ruby
?> module Hello
?>   def say_hello
?>     "hello"
?>   end
>> end

?> class Greeting
?>   class << self
?>     include Hello
?>   end
>> end

>> Greeting.say_hello
=> "hello"
```

This is some exotic syntax. But all that line with `class << self` is doing is opening the singleton class of `Greeting`.

Note that we also made `say_hello` an instance method of the module (instead of a class method).

Why does this work?

When we `include` the module in the target class's singleton class, the module's instance methods become instance method's of the target class's singleton class. This is good. Remember that singleton class is where the class keeps its *class methods*. So in our case, the module's methods become class methods on the target class. Which is what we wanted.

**What is `extend`?**

`extend` is an instance method of `Object`. `Object#extend` is simply a shortcut that includes a module in the receiver's singleton class. We can always do that manually as shown above, but the syntax for it, with the `class << self`, is a bit awkward. So instead we can use `extend`:

```ruby
?> class Greeting
?>   extend Hello
>> end

>> Greeting.say_hello
=> "hello"
```

Here are some facts about `include` and `extend` and how to think about when to use each:

- `include` mixes in the specified module's instance methods as instance methods in the target class.
- `include` is a private method of `Module`. It's intended to be called from within a class/module definition.
- `extend` adds the specified module's methods to the target class's singleton class. And as a result, as class methods on the target class.
- `extend` is a public method of `Object`. And can be used at the 'top level'.
- In addition to classes, we can also `extend` objects in Ruby. If we call `obj.extend(MyModule)`, now `obj` has `MyModule's` methods. But no other instance of the object `obj`'s class has those added methods. This is wild. Those added methods are called the object's singleton methods and we can list them with `obj.singleton_methods`
- One more thing to know about `include` is that `include` adds the module to the target class's ancestor chain, right above the class itself. We can see the list with `MyClass.ancestors`.

To wrap up, we saw a number of different ways to answer the question "what's the difference between `include` and `extend`?". 

The short answer is that `include` adds instance methods, `extend` adds class methods.

A more complete answer involves talking about the singleton classes and opening them up with the exotic `class << self` syntax. 

Note that neither `include` or `extend` are keywords though. They are both methods, like most things in Ruby.
