---
layout: post
title:  "How do Ruby objects and instance variables work?"
---

Ruby is an object oriented language and has classes and objects and methods and instance variables and all those things. But if you are coming from other object oriented languages like Java, there are a few surprising things about how Ruby objects work that are useful to know. 

Here are some interesting things about ruby objects and instance variable and methods to keep in mind when you read/write Ruby code:

## Objects of the same `class` can carry different instance variables. 

In other words, instance variables 'belong to' or 'live in' the object. There is no connection between an object's class and its instance variables.

Imagine we have this class Hello and two objects h1 and h2. h2 has the `@last_name` instance variable but h1 does not.

```ruby
?> class Hello
?>   def initialize
?>     @name = "Annie"
?>     @greeting = "Hello"
?>   end
>> end
 
>> h1 = Hello.new
=> #<Hello:0x00007fa46786a930 @greeting="Hello", @name="Annie">
>> h2 = Hello.new
=> #<Hello:0x00007fa462f059e8 @greeting="Hello", @name="Annie">
 
>> # Add an instance variable to one of the two objects, h2 but not h1

>> h2.instance_variable_set("@last_name", "Smith")
=> "Smith"
>> h2
=> #<Hello:0x00007fa462f059e8 @greeting="Hello", @last_name="Smith", @name="Annie">
>> h1
=> #<Hello:0x00007fa46786a930 @greeting="Hello", @name="Annie">
``` 

So now `h1` and `h2` are objects of the same class `Hello` but have different set of instance variables. What do you think will happen if we add an instance method to `Hello` that *uses* `@last_name`? Let's see.

```ruby
?> class Hello
?>   def greet
?>     puts "#{@greeting}, #{@name} #{@last_name}"
?>   end
>> end

>> h2.greet
Hello, Annie Smith
                                
>> h1.greet
Hello, Annie 
                                
>> h1.instance_variable_get("@last_name")
=> nil
>> h1.instance_variable_get("@blah")
=> nil
```

Nothing crazy. Things just work as expected. Looks like the value of *any* instance variables that have not been assigned is `nil`.

Now let's talk about the instance methods.

## Methods of an object 'live in' the object's class.

So unlike instance variables, *methods* of an object are attached to its class. From the class's perspective they are called *instance methods*. From the object's perspective they are just *methods*. 

We can list the methods of an object by calling the `Object#methods` method on the object or the `Module#instance_methods` method on its class. (these methods are inherited from ruby's `Object` and `Module` classes. We'll talk more about ruby's class hierarchy in a later post).

```ruby
>> class Hello; end

>> h = Hello.new
=> #<Hello:0x00007f843b8c1038>
>> h.methods(false)    # 'false' excludes methods from ancestors
=> []
>> Hello.instance_methods(false)
=> []
>> Hello.instance_methods  # without 'false', many inherited methods
=> 
[:pretty_print,                              
 :pretty_print_inspect,
 ...
 :__send__]
 
>> h.methods == Hello.instance_methods
=> true

?> class Hello
?>   def greeting
?>     "hello"
?>   end
>> end

>> Hello.instance_methods(false)
=> [:greeting]
>> h.methods(false)
=> [:greeting]

```

In the above example, we create an empty class `Hello` and an object `h` and confirm that the object's methods are same as its class's instance_methods. Then we add a method `greeting` and see it appear in the list of object's methods / classes's instance_methods.

Aside: note that in Ruby, we can add methods to a single object as well. In that case, they are not called instance methods or methods though. They are called singleton methods. When add methods to an object, that object has methods that no other object of the same class does. (If that sounds wild to you, you're not alone! We'll get to how that works in a later post. It requires learning about the concept of singleton class (aka eigenclass, aka metaclass)).

One more interesting thing about instance variables.

## Instance variables spring into action when you assign them.

In other words, they don't have to be 'declared' ahead of time, as part of the class definition.

```ruby
>* class Hello
>*   def add_instance_var_person
>*     @person = "Joe"
>*   end
> end

> x = Hello.new
=> #<Hello:0x00007fecc46a3160>
> x.instance_variables
=> []
> x.add_instance_var_person
=> "Joe"
> x.instance_variables
=> [:@person]
```

In the above example, we create a class `Hello` with an *instance method* that defines an *instance variable* `@person`. An object that is an instance of `Hello` that calls the method `add_instance_var_person` will have `@person` instance variable 'attached' to it. But before it calls that method, it's list of instance variables is empty `[]`. 

This implies that another object of `Hello` that has never called the instance method will not have the instance variable `@person`. This goes hand in hand with first fact above. Objects of the same class can have different instance variables 'attached' to them.  

This may all feel surprising. At least it did to me at first.

To wrap up, what is an object? An object is composed of a bunch of instance variables and a link to a class. Object's class carries its instance methods but its instance variables live in the object itself.