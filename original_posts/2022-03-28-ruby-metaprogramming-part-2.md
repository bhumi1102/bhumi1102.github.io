layout: post
title:  "Ruby Metaprogramming Part 2"
---

We continue with more reflection methods in Ruby like `instance_variables`, `local_variables`, `constants` for variables and `public_methods`, `private_methods`, `method_definded?` for methods. 

Note: This is part 2 of metaprogramming in Ruby, in [part 1 we covered evaluating strings and blocks](__GHOST_URL__/ruby-metaprogramming-part-1/) with `instance_eval` and such.

## Reflection Methods for Variables and Constants
Here are some methods for listing the names of all instance variables on an object, all class variables of a class or module and all constants. 

Given this class

```ruby
class Color

  def initialize(name)
    @name = name
  end

  @@sky = "blue"
  TREE = "green"
end
```

we can see `class_variables`, `constants`, and `instance_variables` in action in `irb`

```ruby
> Color.class_variables
=> [:@@sky]

> Color.constants
=> [:TREE]

> Color.new("purple").instance_variables
=> [:@name]
```

In addition to listing these, we can also query and set the value of class and instance variables and constants and check whether one is defined.

```ruby
o = Object.new
o.instance_variable_set(:@greeting, "hello")

> o.instance_variable_get(:@greeting)
=> "hello"

> o.instance_variable_defined?(:@greeting)
=> true
```

## Reflection for Methods
Here we look at methods for listing, querying, invoking, and defining *methods*. Yes very meta indeed.

### Listing methods
Let's see some methods for listing methods of a `String`

```ruby
> s = "hello from codecurious.dev"
=> "hello from codecurious.dev"

> s.methods
=> [:unicode_normalize!,
 :pretty_print,
 :encode!,
 :to_c,
 :unpack,
 :include?,
 :next!,
 :upto,
 :match?,
 :rindex,
 :replace,
 :empty?,
 :eql?,
 :getbyte,
 :setbyte,
 :clear,
 :chr,
 :scrub!,
 :dump,
 :byteslice,
 :scrub,
 :upcase,
 :downcase,
 ...
  ]
```

I made the mistake of pasting the entire output here and that took over my entire editor. `String` has a lot of methods. The above is small sample. 

`public_methods` is the same as `methods`. There is also `private_methods` and `signleton_methods`. We can also check if a method is defined.

```ruby
> String.method_defined? :upcase!
=> true
```

### Invoking methods
We can obtain [Method objects](__GHOST_URL__/ruby-method-object-define_method-and-bind/) by calling `method` an object like `"hello".method(:reverse)` returns a `Method` object bounded to the string "hello". `Method` objects have a `call` method that we can use to invoke the method.

Another way to invoke methods is with `send` method of `Object`. We can invoke instance method as well as class methods with `send`.  The argument is the method name and the rest are passed on as parameters.

```ruby
> "hello".send :upcase
=> "HELLO"

> Math.send(:sin, Math::PI/2)
=> 1.0
```

`send` can invoke any named methods of an object, including private methods. Ruby 1.9 added `public_send` which only invokes public methods. `__send__` is a synonym for `send`.

### Defining methods
We can use `define_method` to define new instance method of a class or a module.  `define_method` takes the name of the method as the first argument and method body as either a `Method` object or a block. `define_method` is private and we need to be inside the class in order to call it.

```ruby
def add_method(c, m, &b)
  c.class_eval {
    define_method(m, &b)
  }
end

add_method(String, :greet) {"Hello, " + self}

"world".greet # prints "Hello, world"
```

`attr_reader` and `attr_accessor` are methods that define new methods for a class. Like `define_method` these are private methods of `Module` and are meant to be used inside class definition. 

We can use `alias_method` to create a synonym name to a method. `alias_method` is private and needs to be used inside a method.

Lastly, `method_missing` of `Kernel` module can be implemented by a given class to allow the class to handle invocation of methods that do not exist. 

`method_missing` is a one of the most powerful and commonly used dynamic programming techniques in Ruby. 
