---
layout: post
title:  "What are procs and lambdas in Ruby"
---

This one is about Ruby. In addition to methods, Ruby has `lambdas` and `procs` as a way of storing pieces of reusable code. 

Both `lambda` and `proc` are instantiated from the same class `Proc` (note the capital P). This is one of the more confusing features of Ruby. But not to worry, we'll clear that up in this post!

First, we'll look at the `Proc` object, different ways to create procs and lambdas, see how to invoke them, and then look at how they differ from each other.

## The Proc Object

Let's start with blocks to get at the motivation for procs and lambdas. Blocks are syntactic structures in Ruby. They are *not* objects. It is possible to create an object that represents a block. 

Depending on how that object is created it's called a *proc* vs. a *lambda*. They are both instances of the `Proc` class. procs have block-like behavior and lambdas have method-like behavior. 

Let's see some code examples!

## Defining Proc and Lambda

So how do we create these procs and lambdas? There are a few ways `Proc.new`, `Kernel.lambda`, `Kernel.proc` and for lambdas there is also a literal syntax `->`.

**1. Proc.new**

This is the normal `new` method of a Ruby class. When `Proc.new` is called with an associated block, it return a proc that represents the block (lower case `p` proc, and *not* a lambda). 

```ruby
>> p = Proc.new { |x,y| x + y }
=> #<Proc:0x00000001113969a0 (irb):33>

>> p.class
=> Proc
>> p.lambda?
=> false
```

Remember that all methods in Ruby can be invoked with a block. If `Proc.new` is invoked within a method that has an associated block, then it returns the block associated with the containing method. 

It can be used like this. I suppose instead of `yield`.

```ruby
def call_proc
  Proc.new.call
end

call_proc { puts "hello" }
=> hello
```

The `.call` will execute the code in the block associated with the invocation of `call_proc` method.

Update: the above *was true* prior to Ruby 3.1 (I [think](https://stackoverflow.com/questions/76658788/ruby-proc-new-in-method-with-implicit-block-no-longer-supported)). When I went to test this code snippet on latest Ruby, `new` fails to find the block and reports `tried to create Proc object without a block`. Apparently this feature was deprecated.

**2. Kernel.lambda**

Another way to create a `Proc` object that is a lambda is the `lambda` method of the `Kernel` module. 

This method acts like a global function. It expects no arguments but must have a block associated with the invocation. And it returns a `Proc` object that is a `lambda` this time (not a `proc`).

```ruby
>> is_positive = lambda { |x| x > 0 }
=> #<Proc:0x00000001114b6b00 (irb):30 (lambda)>

>> is_positive.class
=> Proc
>> is_positive.lambda?
=> true
```

**3. Lambda literal syntax**

This is the `->` that you've probably seen in your Rails application code. It is syntactic sugar for the `lambda` method.

For example, this lambda `increment = lambda {|x| x + 1}`  can be written like this as well `increment = ->(x){x + 1}`. 

```ruby
# same as lambda {|x| x + 1}
>> increment = ->(x){x + 1}
=> #<Proc:0x000000011119aac0 (irb):37 (lambda)>

# same as above, using numbered parameters
>> increment = -> { _1 + 1}
=> #<Proc:0x000000011113f350 (irb):38 (lambda)>
```

The `-> { _1 + 1}` is even more succient. This is using numbered parameters, `_1` refers to the first block parameter (this is not specific to procs or lambdas, works with any old block).

One advantage of doing the `->` syntax is that we can clearly indicate parameters with default values. So we can write `->(x, y=1){x + y}`. Where the 2nd parameter `y` has a default value of 1.

This `lambda {|x, y=1| x + y}` also works btw, just not as easy to read.

This syntax is commonly used for passing lambda as argument to methods or other lambda. 

**4. Kernel.proc**

Including this here for completeness but this one is not as common.

```ruby
>> foo = proc {|x| x + 1}
=> #<Proc:0x000000011121b440 (irb):45>

>> foo.class
=> Proc
>> foo.lambda?
=> false
```

Note: I think prior to Ruby 1.8 `proc` was a synonym for `lambda`! Very confusing. Ruby 1.9 fixes this and makes it a synonym for `Proc.new`. so `proc` *could* be used instead of `Proc.new`. It's shorter but still could be confusing unless you're sure you'll never run Ruby less than 1.9.

Now that we know how to define procs and lambdas, let's move on to how to call them.

## Invoking Procs and Lambdas

Procs and Lambdas are objects not methods. So they cannot be invoked in the same way as methods. Instead we invoke the method `call` on a `Proc` object `p` like `p.call`. This will execute the code in the block associated with the `Proc` object and return the value of the block as return to the `.call`. 

In addition to `call`, there are two other ways that are equivalent.

`p[x,y]` is the same as `p.call(x,y)`. 

`p.(x,y)` is also syntactic sugar for `p.call(x,y)`.

We can invoke a lambda from the previous section like this:

```ruby
>> increment = ->(x){x + 1}
=> #<Proc:0x000000011139d480 (irb):48 (lambda)>

>> increment.call(1)
=> 2

>> increment.(42)
=> 43

>> increment[64]
=> 65

# but not like this
>> increment(8)
(irb):52:in `<main>': undefined method `increment' for main:Object (NoMethodError)

```

Note that we can't call it like a method with `()` as procs and lambdas are objects of the `Proc` class, not methods.

Lastly, let's take a look at how procs and lambdas differ.

## How Lambdas Differ from Procs

A proc is an object form of a block. A lambda behaves more like a method than a block. Calling a proc is like yielding to a block. While calling a lambda is more like invoking a method.

`Proc` class has an instance method `lambda?` that returns `true` for lambda and `false` for proc.

procs and lambdas have different behavior when it comes to the `return` statement and argument passing.

**1. Return statement**

As we know a return statement within a block, not only returns from the block but also returns from the method enclosing the block. Since procs are like blocks, this is true for return statements within a proc as well. It returns from the enclosing method/context.

```ruby
def test
  puts "entering method"
  p = Proc.new {puts "entering proc"; return}
  p.call
  puts "exiting method" # this is never executed
end

>> test
entering method
entering proc
=> nil
```

A return statement in a lambda, on the other hand, return from the lambda itself, *not* from the method that contains the lambda. As we'd expect.

**2. Argument Passing**

procs are invoked with 'yield semantics' and lambda are invoked with method 'invocation semantics'. 

procs turn out to be flexible with the number of arguments they can accept. procs will discard extra arguments, unpack and pack arrays, etc.

lambdas are not flexible in this way. Lambdas must be invoked with precisely the number of arguments they are declared with. 

Alright, that covers it all. We know about the `Proc` class, how to create and invoke procs and lambdas. We know how they differ. And we know what the syntax `->` and `_1` mean.
