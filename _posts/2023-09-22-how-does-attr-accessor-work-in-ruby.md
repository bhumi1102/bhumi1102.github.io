---
layout: post
title:  "How do attr_accessor and attr_reader work?"
---

When we define a `class` in an object oriented language, we typically add a few instance variables to the class. In order to read and write those instance variables outside of the class, we add 'getter' and 'setter' instance methods to the class (you may have seen these in other programming languages like C++ or Java). In Ruby, it can look something like this: 

```ruby 

class Person
  def name
    @name
  end
  def name=(your_name)
    @name = your_name 
  end 
end 

person = Person.new
person.name = "Jim" 
=> "Jim"
person.name
``` 

However, we don't typically see code that looks like that in Ruby. That's because we don't need to add these getters/setters manually. Ruby does that for us automatically. More specifically it dynamically adds those methods to our class using metaprogramming at runtime (We'll cover why this is possible in a later post. Don't worry about it for now). All Ruby needs to know is the name of the instance variables we wish to set up. And it can add methods matching those names. 

The method `attr_reader` adds a getter only, while `attr_accessor` adds a getter and setter both. (There is also `attr_writer` to add a setter method only.) 

With `attr_accessor`, the above code becomes: 

```ruby
class Person
  attr_accessor :name
end
``` 

Note that `attr_accessor :name` is not some special syntax in ruby. It is a method call (on the `Module` class) with one parameter - the symbol `:name`. Rewriting it with parens would look like this:

```ruby 
class Person
  attr_accessor(:name)
end 
``` 

We can see that the appropriate instance methods are added with 

```ruby
> Person.instance_methods 
=> [:name=, :name, ... ] 
```

That's all there is to it. You can write `attr_accessor` in your Ruby class and never worry about manually adding the boilerplate code for reading and writing the value of an instance variable. 

## Extra 
(If you are new to programming or to Ruby/Rails, the below may not fully make sense and that's okay. I still encourage you to read it.) 

- If you're familiar with `ActiveRecord` in `Rails`, you know that we're able to access the column names in our table as instance methods on our ActiveRecord object (If we have table named `Person` with a column called `name`, we do [http://p.name](https://t.co/KIAVRpJsCa) in our Rails code, assuming `p` is an instance of `Person`). But notice that we can do this *without* having to even add `attr_*` in our ActiveRecord class. How is that possible? This is because Rails can infer the names of methods from the column names in our migrations and schema.rb file. 

- How do all Ruby classes have access to these special methods `attr_accessor`? Where do they come from? All Ruby `Classes` are `Modules`. Meaning they are subclass of `Module` and *inherit* methods from `Module`. The ancestor chain is `Class` --> `Module` --> `Object` --> `BasicObject`. And below we can see that these `attr_*` methods are coming from Module (and not Object or something else). 

```ruby
> Class.superclass 
=> Module 

> Module.instance_methods.grep(/^attr/)
=> [:attr, :attr_reader, :attr_writer, :attr_accessor] 

> Class.instance_methods.grep(/^attr/) 
=> [:attr, :attr_reader, :attr_writer, :attr_accessor] 

> Object.instance_methods.grep(/^attr/)
=> [] 
```


