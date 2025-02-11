layout: post
title:  "How &: Works in Ruby"
---

Chances are you've seen and written code like `users.map(&:first_name)` in ruby or rails applications many times. If you're new to Ruby, you may have wondered about this strange syntax. Let's demysify `&:` and explain exactly how it works.

First we start with blocks and how to pass block arguments to methods.

### Passing block arguments to methods and the `&`

We can think of a Block as a chunk of ruby code associated with method invocation. We know that any method invocation may be followed by a block and the method can invoke the code in that block with a `yield` statement.

Here is an example. This method generates a sequence of `n` numbers offset by a constant `c` and passes them to the block.

```ruby
def seq(n, c)
  i = 0
  while (i < n)
    yield i + c
    i += 1
  end
end
```

And we can invoke that method like this

```ruby
  seq(5, 2) { |x| puts x }
```

In this case the block we passed to the method is anonymous. What if we wanted to explicitly refer to the block within the method? We can add a named block argument and precede it with an `&`. If we do this, the block will be converted to a `Proc` object. Then we can use the `call` method of the `Proc` object instead of the keyword `yield` in this case (although `yield` works too). 

```ruby
def seq(n, c, &b)
  i = 0
  while (i < n)
    b.call(i + c)
    i += 1
  end
end
```

This changes the method *definition*. We still *invoke* the method the same way as before `seq(5,2) { |x| puts x }`. So when would we use the `&` with method invocation as well?

When `&` is used before a `Proc` object in a method invocation, it treats the `Proc` as if it was an ordinary block following the invocation. Consider the following code that sums up the numbers in an array.

```ruby
a = [1, 2, 3, 4, 5]
a.inject(0) { |total, n| total + n }
```

We could be more explicit about using a `Proc` object for the block.

```ruby
a = [1, 2, 3, 4, 5]
summation = Proc.new { |total, n| total + n }
a.inject(0, &summation)
```

And that code snippet is equivalent to the one above.

One more thing to note. In a method invocation, an `&` usually appears before a `Proc` object. However, it's allowed before any object that has a `to_proc` method. For example `Method` object and `Symbol` both have `to_proc` after Ruby 1.9. So that is why we see code like `users.map(&:first_name)`. 

Now let's work out exactly how `&:first_name` above works.

### How does the `&:` thing work?

This is where `to_proc` method of `Symbol` class comes in. So `users.map(&:first_name)` translates to `users.map { |u| u.first_name }`

We can see that that translation is done with a **combination of a symbol, the method `Symbol#to_proc`, implicit class casting, and `&` operator**. 

Because..

1. If we put `&` in front of a `Proc` instance in the argument position, that will be interpreted as a block. 
2. If we put something other than a `Proc` instance with `&`, implicit class casting will try to convert that to a `Proc` instance using `to_proc` method defined on that object if there is any. In case of a `Symbol` there is a `to_proc` method.

One last thing. A subtle point to note: all method names are available by their Symbol names as well. You can call a method on an object like `1.to_s and 1.send(:to_s)`. So really `(1..10).each(&:to_s)` is equivalent to `(1..10).each { |x| x.send(:to_s) }`. The symbol is passed as an argument to the send() method.
