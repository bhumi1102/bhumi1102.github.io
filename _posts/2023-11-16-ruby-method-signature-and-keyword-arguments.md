---
layout: post
title:  "How do `*`, `**`, `&` work in method signatures? Also keyword arguments"
---

These characters, `*`  and `**` and `&`, are quite common in Ruby code. But their meaning is overloaded and depends on where they are used.

This is a quick post to show examples of `*`, `**`, and `&` in method signatures, method invocations, and as a general operator. 

While we're on the topic of method signatures, we'll cover keyword arguments as well (vs. hash arguments and positional arguments). 

Note: This post is intended to be ruby/rails newcomer friendly. If you're an experienced ruby developer, you likely already know this.

## Method Signature & Invocation

`*` and `**` allow us to work with a variable number of arguments and `&` allows us to work with blocks in a method.

**The so called 'splat' operator, `*` , collects multiple arguments into an Array.** Let's see a code example: 

```ruby
# defining a method with *
?> def add_numbers(*nums)
?>   sum = 0
?>   nums.each { |num| sum += num }
?>   sum
>> end

>> add_numbers(1,2,3)
=> 6

# calling a method with *
>> my_array = [2, 4, 8, 16]
>> add_numbers(*my_array)
=> 30

>> add_numbers(my_array)
(irb):37:in `+': Array can't be coerced into Integer (TypeError)
```

In the method signature `*nums` implies that any number of arguments will be collected in an Array and then `nums` can be treated as an Array within the method body. 

When calling the `add_numbers` method, we can pass it an array we already have with `add_numbers(*my_array)` and the elements of `my_array` will get 'spread out' into 4 arguments. If we leave out the `*` in `*my_array` we can a TypeError, as the method signature expects variable number of arguments, not an Array.

**The 'double splat' operator `**` collects multiple key/value arguments into a Hash**. The Hash can be accessed inside the method as usual. 

We can call the method if we already have a Hash lying around by sticking a `**` in front of the name.

```ruby
# defining a method with **
?> def user(name, **params)
?>   puts "name is #{name}. Age is #{params[:age]}. Location is #{params[:city]}"
>> end

>> user("ruby", age: 28, city: "san diego")
name is ruby. Age is 28. Location is san diego

# calling a method with **
>> other_info = {age: 28, city: "san diego"}
>> user("ruby", **other_info)
name is ruby. Age is 28. Location is san diego
```

Here we call the `user` method with `other_info` Hash by using `**` `user("ruby", **other_info)`.

And the last one. **`&` captures a block and gives it a name so that we can refer to a block inside the method.** More precisely it converts the block into a `Proc` object, which is where the `call` method comes from, in `block.call(num)`.

```ruby
?> def process_numbers(numbers, &block)
?>   numbers.each { |num| block.call(num) }
>> end

>> my_numbers = [1, 2, 3, 4, 5]
>> process_numbers(my_numbers) { |num| puts num * 2 }
2
4                                      
6                                      
8                                      
10
```

Note that all methods in Ruby can be invoked with a block, even if there is no `&` argument. In that case, we call the block using `yield` from the method.

```ruby
?> def process_numbers(numbers)
?>   numbers.each { |num| yield(num) }
>> end
```

It's a good idea to check whether a block is passed to a method or not using `block_given?`, otherwise we'll get a LocalJumpError when we `yield`.

```ruby
>> process_numbers(n)
(irb):63:in `block in process_numbers': no block given (yield) (LocalJumpError)
```

So that's all about `&` and working with blocks in methods.

## Other Operations

As I said, `*`, `**`, and `&` are common in Ruby code and their meaning depends on the context and how they are used.

For example, `*` can of course be used for multiplication and `**` is used for raising a number to a power and `&` is used for bitwise AND operation, as well as array intersection in Ruby.

Side Note: When we do `2 ** 3` to get 2 raised to the power of 3. But `**` is _not_ a language operator (as it is in python). It's a method! `2 ** 3` is the same as `2.send(:**, 3)`

The point is, if you see `&` in a bitwise operation, it has nothing to do with blocks or passing blocks to methods. And when you see `*` or `**` in a method signature or invocation, it has nothing to do with math.

We saw that `*`, `**`, and `&` have special meaning when used in a method signature and method invocation. Now let's see examples of *keyword arguments* vs. hash arguments. 

## Keyword Arguments

Let's say we need to pass several parameters to a method. We can define and call a method like this: 

```ruby
?> def tag(name, id, attributes, content)
?>   # create some HTML tag
>> end

>> tag("Name", "Id", "Attr", "Content")
```

In that case, we have to remember the order of arguments when we call the method (these are called *positional arguments*). Otherwise the method won't work. This is definitely error prone. There's gotta be a better way.

If we pass a Hash with keys/values, we don't have to worry about the order of arguments. Like this:

```ruby
def greet(params)
  puts "Hello, #{params[:name]}! You are #{params[:age]} years old."
end

>> greet({ name: "Jane", age: 30 })
Hello, Jane! You are 30 years old.
```

In this case, we can refer to the key names inside the method definition. But by looking at the method *signature* we don't have a way of knowing the names of the keys the method expects. It can be better. 

Ruby 2.0 introduced keyword arguments. **Keyword arguments syntax gives us a way to be even more explicit by using named parameters in a method definition and method invocation.**

```ruby
?> def greet(name:, age:)
?>   puts "Hello, #{name}! You are #{age} years old."
>> end

>> greet(name: "Jane", age: 30)
Hello, Jane! You are 30 years old.
```

Keyword arguments can have default values as well. They can be set directly in the method signature `def greet(name:, age: 18)`.

That's all for this one. This was a quick review of passing Arrays, Hashes, Blocks, and keyword arguments while defining and calling methods. 
