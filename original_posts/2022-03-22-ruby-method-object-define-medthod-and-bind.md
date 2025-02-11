layout: post
title:  "Ruby Method Object, define_method and bind"
---

Ruby methods are executable language constructs. They are not objects. But we can get objects that represents methods with the `Object` class's `method` method. 

Note: This is similar to how blocks are language syntax but not objects in Ruby and we can get object versions of blocks in the forms of [procs and lambdas]().

So the `Object#method` takes a method name - either string or symbol - and return an `Method` object. This object represents the named method on the receiver. 

```ruby
l = "hello".method(:length)
puts l.call #will return 5
```

Here we get a Method representing the `length` method on the `String` call, bound to the string "hello". And we can call this method with `Method#call`.

Note: We can use `public_method` which works the same as `method` but ignores protected and private methods.

### Method to Proc and Proc to Method

`Method` objects work very much like [`Proc` objects]() and can be used in place of them. `Method` even has a `to_proc` method to convert it to `Proc` if necessary. Here is an example where the method is passed in place of a proc with an `&`:

```ruby
def square(x)
  x * x
end
puts (1..10).map(&method(:square))
```

We can also go in the other direction. Instead of getting `Method` objects and converting them to `Proc`, we can create a `Method` from blocks and procs using `define_method` of the `Module` class.

`define_method` expects a `Symbol` as an argument and create a method with that name using the associated block as the method body like this `define_method(:say_hello) { puts "hello" }`.  This is the same as if we had done this:

```ruby
def say_hello
  puts "hello"
end
```

### Unbounded Method objects and bind

So in the examples above we get `Method` objects that are tied to an instance of a class that contains the method. These `Method` objects have a receiver.  Can we get a `Method` object that was not bounded to an instance? 

Turns out we can. It would be of a class called `UnboundMethod` though. And we cannot actually call this `UnboundMethod` as it wouldn't make sense. For example 

```ruby
unbounded_power = Fixnum.instance_method("**")
```

creates a `unbounded_power` method that is of type `UnboundMethod`. And in order to actually invoke this method we can have to `bind` it to something

```ruby
power_of_2 = unbounded_power.bind(2)
puts power_of_2.call(5) # prints 32
```

The `power_of_2` is a `Method` object now and we can use its `call` method.

Lastly, `Method` has some other handy methods  `name`, `owner`, and `receiver` that work as below.

```ruby
> power_of_2.name
=> :**

> power_of_2.owner
=> Integer

> power_of_2.receiver
=> 2
```

That's all about `Method` objects in Ruby and how they compare to `Procs`.
