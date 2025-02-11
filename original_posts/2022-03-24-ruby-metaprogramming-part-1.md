layout: post
title:  "Ruby Metaprogramming Part 1"
---

`define_method`, `Kernel.eval`, `instance_method`, `instance_eval`, `class_eval`, `instance_exec`, `send`, `bind`. Have you seen Ruby programs with these methods or haven't been sure what each of them do exactly?

By the end of this post you will be able to confidently read Ruby code that contains these metaprogramming constructs. 

---

Let's zoom out and define some terms so that we are on the same page.

Using all those methods is called doing *metaprogramming* and in Ruby we are using the *reflection* API. 

*Reflection* (aka introspection) just means a program can examine its own state and structure. And also modify it at runtime. For example, a Ruby program can define new classes and new methods at runtime, get a list of methods of a given class, get a list of all objects of a given class currently known to the interpreter, etc.

*Metaprgramming* in Ruby is a set of techniques for extending Ruby's syntax in a way that makes programming easier. Metaprogramming is used in defining Domain Specific Languages (DSLs) in Ruby for example.

*Domain Specific Language* is basically when we use method invocations and blocks as if they were task specific *keywords* in an extension of the language. Testing library `RSpec` is an example of a DSL.

Now let's see all of the things we can do with Ruby's 'meta' or reflection methods.

## Ruby Reflection API 

Ruby reflection methods like `define_method`, `eval`, `instance_method`, `instance_eval` and such are defined by either `Kernel`, `Object`, or `Module`. 

Let's see how each of them works with some code.

### Types and Classes

 `o.instance_of? c` check whether an object is an instance of class

 `o.respond_to? name` check whether an object has a method

`o.class` and `o.superclass` return the class and superclass of an object.

`o.is_a? c` returns true if `o` is an instance of `c` or any of its subclasses.

Get all the ancestors of a class

```shell
> String.ancestors

=> [String, Comparable, Object, PP::ObjectMixin, Kernel, BasicObject]
```

We can even define new classes and modules at runtime. The code below defines a new module, a new class, then a subclass that includes the module.

```ruby
my_module = Module.new
my_class = Class.new
my_subclass = Class.new(my_class) {
  include my_module
}
```

### Evaluating Strings and Blocks

Before we get to what `instance_eval` and `class_eval` do, we need to learn about the `Binding` object. 

`Binding` object represents the state of Ruby's variable bindings at some moment. So if we write

```ruby
n = 40
eval "n + 2"
```

the value of `n` is part of the binding for `eval`. By the way, `eval` is from `Kernel` and it can evaluate any string of valid ruby code. It's not something we'd use in our regular programs but it's powerful.

Okay back to `Binding`.

`Kernel.binding` returns the bindings in effect at a given location in our program. And we can pass a `Binding` object as the second argument to `eval`. In that case, the string we pass will be evaluated *in the context* of those bindings. Here is an example

```ruby
class Pizza
  def initialize(type)
    @type = type
  end
end

p = Pizza.new("margherita")
eval ("@type", p.bindings) # returns "margherita"
```

#### Proc Object and Bindings

The `Proc` object defines a `binding` method that returns a `Binding` object. That `Binding` object represents the binding in effect for the body of that `Proc`.

`eval` method allows us to pass in a `Proc` object instead of a `Binding` object as the second argument.

#### instance_eval and class_eval

There are two main differences between the regular `eval` from `Kernel` and `instance_eval`/`class_eval`. 

1. `instance_eval` and `class_eval` can accept a block of code to evaluate, instead of just strings.
2. The context in which they evaluate the specified string or block. `instance_eval` evaluates the code in the context of the specified object and `class_eval` does in the context of a specified module.

`o.instance_eval("@x")` returns the value of `o`'s instance variable `@x`.

Here is an example that involves defining methods on an object.

```ruby
String.class_eval {
  def my_size
    size
  end
}

String.instance_eval {
  def secret_knock
    "Say friend and enter"
  end
}
```

The first defines an instance method called `my_size` which is an alias for String's existing `size` method. The second one actually defines a singleton or class method `String.secret_knock`. (If you noticed that the name of the meta method and the kind of method they define seem 'backwards', you're right. I was surprised too.)

There are two other methods (starting Ruby 1.9) `instance_exec` and `class_exec`. They only evaluate blocks (not string) and they accept arguments and pass them to the block. So the block code is evaluated in the context of the specified object, with parameter from the outside.


That is all for Part 1. We looked at Ruby reflection methods for types, classes, and modules as well as evaluating string and blocks. In Part 2, we look at metaprogramming around variables, constants and methods.
