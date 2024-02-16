---
layout: post
title:  "What are class instance variables in Ruby?"
---

In other object oriented programming languages like Java, we generally have instance variables and class variables. But in Ruby, we also have *class instance variables*? What are those, and how are they different from *instance* variables and *class* variables?

The name *class instance* variable sounds puzzling, but the concept is pretty simple and consistent with the fact that everything in Ruby is an object. 

This will be short one. Let's start with some code:

```ruby
?> class User
?>   @greeting = "hello"
?>   def an_instance_method
?>     @name = "bhumi"
?>   end
>> end
```

Notice the variable `@name` inside an instance method of the class `User.` And another variable `@greeting` which is outside of any methods but inside the `User` class definition. They both start with an `@`. But they don't feel the same. What will the Ruby interpreter do with each?

This is what the interpreter does. It assumes that all instance variables, the ones starting with `@`, belong to `self`. (`self` is the 'current object' in Ruby. I will write a later post that spells out the value of `self` and how it changes in different context).

The value of `self` inside an instance method is the object (aka the receiver) that called the method. In this case, an object of the class `User`. So `@name` is an ***instance variable*** of that object. 

The value of `self` inside a class definition is the class itself. And remember classes are just objects. They are objects of the class `Class`. So the `@greeting` variable is an instance variable of the class `User` that we're defining. `@greeting` is called a ***class instance variable***.

We can see this by calling the `instance_variables` method on both the object `u` and the class `User`.

```ruby
>> u = User.new
>> u.an_instance_method
=> "bhumi"

>> u.instance_variables     # instance variable
=> [:@name]

>> User.instance_variables  # class instance variable
=> [:@greeting]
```

To summarize, **instance variables of the class are different from the instance variables of that class's objects**. In our example, `@name` is an instance variable and `@greeting` is a class instance variable. And they are different.

Also note that *class instance variables* can only be accessed by the class itself, not by subclasses or instances of the class.

**Class Variables**

So what about plain class variables, does Ruby have those? Ruby does have class variables. They start with `@@` like `@@a_class_variable`. They are similar to things like static fields in languages like Java. 

As we can see below, unlike *class instance variables*, **class variables** can be accessed from regular instance methods.

```ruby
?> class MyClass
?>   @@my_class_var = 42
?>   def an_instance_method
?>     @@my_class_var
?>   end
>> end

>> MyClass.class_variables
=> [:@@my_class_var]

>> c = MyClass.new
>> c.an_instance_method
=> 42
```

And from subclasses:

```ruby
?> class MySubClass < MyClass
?>   def a_subclass_method
?>     @@my_class_var
?>   end
>> end

>> MySubClass.new.a_subclass_method
=> 42
```

Side note: class variables don't quite 'belong to' the class. They can be surprising in this way. In the older versions of Ruby, you could access variables with `@@` at the top-level, and accidentally mixup that variable with a class variable by the same name.

But now Ruby will give us a `RuntimeError` if we try to access class variables with `@@` at the top level. This is good. It makes working with class variables less surprising.

```ruby
>> @@some_variable = "boo"
(irb):1:in `<main>': class variable access from toplevel (RuntimeError)
```

To wrap up, here are some facts to take away:

- *Instance variables* belong to the objects of the class and *class instance variables* belong to the class, which is an object itself of the class `Class`. They are different.
- *Class variables* are different from *class instance variables* in that *class variables* can be accessed from regular instance methods of the class and by subclasses. Class instance variables, on the other hand, can only be accessed by the class itself, not by subclasses or instances of the class.


P.S. In addition to these variables, we also have constants. Constants in Ruby are interesting. For example class and module names are just constants. Here are [two](https://theleafnode.com/how-do-constants-work-in-ruby/) [posts](https://theleafnode.com/does-using-freeze-prevent-changing-the-value-of-ruby-constants/) related to constants. 