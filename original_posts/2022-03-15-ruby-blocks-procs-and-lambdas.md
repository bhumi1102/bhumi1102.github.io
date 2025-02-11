layout: post
title:  "Difference Between Ruby Blocks, Procs, and Lambdas"
---


If you're new to Ruby or if you've been using Ruby for a while but want to understand procs and lambdas a bit deeper, this post is for you. 

We will look at the `Proc` object, four ways to create procs and lambdas, see how to invoke them, and look at how they differ from each other.

### The Proc Object
Let's start with blocks to get at the motivation for procs and lambdas. Blocks are syntactic structures in Ruby. They are not objects. It is possible to create an object that represents a block. Depending on how that object is created it is called a *proc* or a *lambda*. 

Procs have block-like behavior and lambdas have method-like behavior. Here is the slightly confusing part. Both *procs* and *lambdas* are instantiated from the same class called `Proc`. yup.

Aside: the word 'lambda' comes from a branch of math called 'lambda calculus'. We can think of it as a word that refers to functions that can be manipulated as objects.

So how do we create these procs and lambdas? There are 3 ways `Proc.new`, `Kernel.lambda`, `Kernel.proc` and for lambdas there is also a literal syntax (introduced in Ruby 1.9).

### Defining Proc and Lambda
Both procs and lambdas are a variety of the same class `Proc`.  Which one we create depends on how we create them. Remember procs behave like blocks and lambdas behave like methods.

**1. Proc.new**

This is the normal `new` method that most classes support. When `Proc.new` is called with an associated block, it return a proc that represents the block. 

```ruby
p = Proc.new { |x,y| x + y }
```

If `Proc.new` is invoked within a method that has an associated block, then it returns the block associated with the containing method. This is a bit strange but it can be used like this. I suppose instead of `yield`.

```ruby
def call_proc
  Proc.new.call
end
```

The `.call` with execute the code in the block associated with the invocation of `call_proc` method.

**2. Kernel.lambda**

The `lambda` method of the `Kernel` module acts like a global function. It expects no arguments but must have a block associated with the invocation. And it returns a `Proc` object that is a lambda (not a proc). Yeah, I know, a little confusing.

```ruby
is_positive = lambda { |x| x > 0 }
```

**3. Kernel.proc**

tl;dr don't use this. Why? 

Because in Ruby 1.8 it was a synonym for `lambda`! (We know naming is hard but come on). Ruby 1.9 fixes this and makes it a synonym for `Proc.new`. so `proc` *could* be used instead of `Proc.new`. It's shorter but still could be confusing unless you're sure you'll never run Ruby less than 1.9.

**4. Lambda literal syntax**

This is a wild one. Adding this syntax to Ruby was controversial. I have not seen this much in the wild. But for the sake of completeness, here is the deal with lambda literal syntax. 

This lambda `next = lambda {|x| x + 1}`  can be written like this as well `next = ->(x){x + 1}`. 

The keyword lambda becomes the ->, parameter is pulled out of || inside the block and put into () outside.

One advantage of doing this is that parameters can have default values. So we can write `->(x, y=1){x + y}`. Where the 2nd parameter `y` has a default value of 1.

It's a weird looking syntax (and reminds me of JavaScript arrow functions) but it's succinct so it could be helpful for passing lambda as argument to methods or other lambda. 

Now that we know how to define procs and lambdas, let's move on to how to call them.

### Invoking Procs and Lambdas

Procs and Lambdas are objects not methods. So they cannot be invoked in the same way methods are. Instead we invoke the method `call` on a `Proc` object `p` like `p.call`. This will execute the code in the original block and return the value of the block as return to the `.call`. 

In addition to `call`, there are two other ways that are equivalent.

`p[x,y]` is the same as `p.call(x,y)`. 

`p.(x,y)` is also syntactic sugar for `p.call(x,y)`.

Here is me trying this out in the console:

```shell
> is_positive = lambda {|x| x > 0}

=> #<Proc:0x00007f89e769e3f8 (pry):22 (lambda)>

> is_positive.lambda?

=> true

> is_positive(4)

NoMethodError: undefined method `is_positive' for main:Object

Did you mean? is_a?

from (pry):24:in `__pry__'

> is_positive.call(4)

=> true

> is_positive.(4)

=> true

> is_positive[4]

=> true
```

So yeah. We can't call it like a method as procs and lambdas are objects of the `Proc` class, not methods.

Lastly, let's take a closer look at how procs and lambdas differ.

### How Lambdas Differ from Procs

A proc is an object form of a block. A lambda behaves more like a method than a block. Calling a proc is like yielding to a block. While calling a lambda is like invoking a method.

`Proc` class has an instance method `lambda?` that returns `true` for lambda and `false` for proc.

**Return Statement**

As we know a return statement within a block, not only returns from the block but also returns from the method enclosing the block. Since procs are like blocks, this is true for return statements within a proc as well. It returns from the enclosing method.

```ruby
def test
  puts "entering method"
  p = Proc.new {puts "entering proc"; return}
  p.call
  puts "exiting method" # this is never executed
end
```

A return statement in a lambda, on the other hand, return from the lambda itself, not from the method that contains the lambda.

**Argument Passing to procs and lambdas**

procs are invoked with 'yield semantics' and lambda are invoked with method 'invocation semantics'. 

procs turn out to be flexible with the number of arguments they can be invoked with. procs will discard extra arguments, unpack and pack arrays, etc.

lambdas are not flexible in this way. Lambdas must be invoked with precisely the number of arguments they are declared with.  

Lastly, learning more about procs and lambdas in ruby helps demystify [the `&:` syntax in ruby](__GHOST_URL__/how-ruby-ampersand-colon-works/). You can read more at that link. 

Hope that makes blocks and procs and lambda in Ruby clear. 
