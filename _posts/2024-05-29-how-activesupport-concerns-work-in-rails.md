---
layout: post
title:  "How ActiveSupport Concerns Work in Rails"
---

This one is about `ActiveSupport::Concern` from Rails. Moreover, it's about how Ruby facilitates organizing and binding code (components/modules) together. 

Ruby uses "mixin" for organizing code and sharing functionality. A "mixin" is a module that you can include in classes to add shared functionality without using inheritance. Basically, when we use `include` and `extend`, it's referred to as "mixin" (most languages use inheritance hierarchies for this).

We'll explore the `ActiveSupport::Concern` module, which is prevalent in the Rails source code and it's how Rails binds components of the framework together.

> *Is `ActiveSupport::Concern` too clever for its own good? That's up to you to decide. Some programmers think that Concern hides too much magic behind a seemingly innocuous call to `include`, and this hidden complexity carries hidden costs. Other programmers praise Concern for helping to keep Rails' modules as slim and simple as they can be. Whatever your take on `ActiveSupport::Concern`, you can learn a lot by exploring its insides.*
                       -- Metaprogramming Ruby 2

Aside: check out Metaprogramming Ruby book, chapter 10, if you want to go deeper into this topic. The example below are from there.

So let's learn things about Ruby and Rails by exploring `ActiveSupport::Concern` - what problem does it solve and how it came to be. 

You've likely seen code like this in your application:

```ruby
module Authorization
  extend ActiveSupport::Concern
  ...
end
```

What does adding `ActiveSupport::Concern` to a module do? It makes it so that including that module (also know as a "Concern") into a target class will give the target class both instance methods *and* class methods from that `Authorization` module.

In order to make sense of that last sentence, we need to review the difference between `include` and `extend` in Ruby.

## The Difference Between `include` and `extend`
Let's say we have a module with a useful method and we want to add that method to our class:

```ruby
module Hello
  def self.say_hello
    "hello"
  end
end
```

We want to use `say_hello` method in the `Greeting` class, so we can `include` the module `Hello` and use it, right? 

```ruby
class Greeting
  include Hello
end

>> Greeting.say_hello
(irb):28:in `<main>': undefined method `say_hello' for Greeting:Class (NoMethodError)
```

No, because `say_hello` is a *class* method. When a class includes a module, it gets the module’s _instance_ methods, but *not* the class methods.

-   `include` mixes in the specified module’s *instance methods* as instance methods in the target class.
-   `extend` adds the specified module’s methods to the target class’s singleton class. And as a result, as class methods on the target class.

The above is from a [previous newsletter post](https://blog.theleafnode.com/include-vs-extend-in-ruby/) I wrote on the topic, check it out for more.

So what does this have to do with `ActiveSupport::Concern`? Let's look at what problem it's solving next.

Note: The implementation of `ActiveSupport::Concern` is metaprogramming heavy. It twists and bends the Ruby object model. So before we look how it's implemented, we'll look at how it's used, which is more straightforward.

## What Problems do "Concerns" Solve?
As we saw above, `include` adds instance methods and `extend` adds class methods. What if wanted a way to add both instance methods and class methods in one go?

The `ActiveSupport::Concern` module provides that functionality. It also makes it easy to add that behavior into modules you define in your application. Meaning it allows you to define modules such that when you include them in a target class, that target class will get both instance and class methods from your module. 

How? We can do that by adding `extend ActiveSupport::Concern` to our module. That's it. Such a module is called a "Concern". We include Concerns into a target class to add the Concern's instance methods and class methods. That's a lot of words, here's some code:

```ruby
module MyConcern
  extend ActiveSupport::Concern

  def an_instance_method
    "an instance method"
  end

  module ClassMethods
    def a_class_method
      "a class method"
    end
  end
end

# A class that includes the Concern
class MyClass
  include MyConcern
end

# Both instance and class methods
>> MyClass.new.an_instance_method
=> "an instance method"

>> MyClass.a_class_method
=> "a class method"
```

Okay that's neat. But what did Rails do before this concept of Concerns existed?

## Rails Before Concern
There was this "include-and-extend" trick: when a module is included in a target class, the `included` hook method is run, which essentially adds an `extend` to the target class to mixin the class methods as well.

```ruby
module ActiveRecord
  module Validations
    # Hook method
    def self.included(base)
      base.extend ClassMethods
    end

    module ClassMethods
      def valid?
        #...
      end
    end
  end
end
```

If this feels a bit convoluted, because it is. The same effect can be achieved by adding the `extend` line directly, in addition to the `include` in the target class. So it is worth asking if the added complexity is worth it for removing this one line? 

In addition to the complexity, this trick also had another issue with chained inclusions. It required keeping track of if a module was included as a 'first-level', 'second-level', etc. To solve this problem of chained inclusions more elegantly `ActiveSupport::Concern` was crafted.

## Concern Implementation: `Module#append_features`

The implementation of `ActiverSupport::Concern` involves metaprogramming. It overrides a core Ruby method named `append_features`. Inside `append_features` is where module inclusion actually happens in Ruby. 

For example, we can break `include` if we override `append_features` to do nothing:

```ruby
module M
  def self.append_features(base)
    # Overriding to do nothing
  end

  def my_method
    "this method will not get included since we broke append_features"
  end
end

class C
  include M
end

>> C.ancestors
=> [C, Object, PP::ObjectMixin, Kernel, BasicObject]

>> C.new.my_method
(irb):22:in `<main>': undefined method `my_method' for #<C:0x00007f952c615ff8> (NoMethodError)
```

`ActiveSupport::Concern`, however, overrides `append_features` and does appropriate things. That's a glimpse of a core idea in the implementation, you can find the rest of the [implementation in the rails codebase](https://github.com/rails/rails/blob/main/activesupport/lib/active_support/concern.rb#L112).

Before wrapping up, I want to share one last high level thought, about programming: even though I have traced this implementation and understood the concepts before, I don't (need to) think about it on a day-to-day basis. Therefore, I don't think I can articulate the above at the drop of a hat. The details of how `ActiveRecord::Concern` is implemented are abstracted away from us as Rails developers. We can simply add `extend ActiveSupport::Concern` to our modules and use the functionality. This is the power of abstraction (which can be a double-edged sword though, hidden complexity, hidden costs, etc.). 

Hope you enjoyed this exploration of `ActiveSupport::Concern` in Rails.
