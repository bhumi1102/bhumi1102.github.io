---
layout: post
title:  "How do constants work in Ruby?"
---

When you hear the word a `constant`, what comes to mind? What would you tell someone what a `constant` is, in a programming language?

A `constant` is a variable whose value cannot be changed once assigned. Right? This is true for most programming languages, in Ruby there is more to the story of `constants`.

First, you may be surprised to see the following:

```RUBY
>> FUN_CONSTANT = "hello"
=> "hello"
>> FUN_CONSTANT = "bye"
(irb):66: warning: already initialized constant FUN_CONSTANT
(irb):65: warning: previous definition of FUN_CONSTANT was here            
=> "bye"                                                                   
>> FUN_CONSTANT
=> "bye"
```

The Ruby interpreter does not enforce the 'constantcy' of constants. The compile issues a warning when you try to reassign a constant but the reassignment still works. (there is a method `Object#freeze` in ruby. Calling `FOO = "hello".freeze` will make it so that we get a runtime error if we try to modify the *value* "hello". However, the constant `FOO` can still be reassigned to a totally different value. It is a bit nuanced, I will cover `freeze` in a later post) 

An aside: about some Ruby conventions. Any variable that begins with a capital letter is a `constant`. Class names and Module names are `constants` also. The convention is to use snake_case with capital letters for regular `constants` `LIKE_THIS`. Class and Modules names are written in PascalCase case `LikeThis`.

**So if we can change the value of a constant, how is a constant different from a variable?**

`constants` are different from variables in their scope or where in the program they're visible.

Unlike variables that are visible locally, `constants` have the visibility of global variables in a sense. **The file system on your computer is a good analogy for developing an intuitive sense for the scope of `constants` in Ruby**.

> You might be surprised to learn that a Ruby constant is actually very similar to a variable - to the extent that you can change the value of a constant...
> If you can change the value of a constant how is a constant different from a variable? The one important difference has to do with their **scope**. The scope of constants follows its own special rules.
                        - *Metaprogramming Ruby 2*

We can visualize all `constants` in a program to be arranged in a tree like structure similar to a file system. Where modules (and classes) can be thought of as *directories* and regular `constants` can be thought of as *files*.

```ruby
?> module MyModule
?>   FUN_CONSTANT = "ruby is fun"
?>   class MyClass
?>     FUN_CONSTANT = "this is NOT the same constant as above"
?>   end
>> end

>> MyModule::FUN_CONSTANT
=> "ruby is fun"
>> MyModule::MyClass::FUN_CONSTANT
=> "this is NOT the same constant as above"
```

We can have multiple `constants` with the same name because they are at different 'levels' or 'paths'. Similar to a file system, where we know we can have multiple files with the same name as long as they live in different directories.

`constants` are uniquely identified by their paths. 

If we are sitting deep inside a tree of modules and classes, we can use leading `::` to access an outer, root-level constants. Like this:

```ruby
>> MY_CONSTANT = "a root-level constant"

?> module Hello
?>   MY_CONSTANT = "a constant inside Hello"
?>   MY_CONSTANT    # => "a constant inside Hello"
?>   ::MY_CONSTANT  # => "a root-level constant"
>> end

>> ::MY_CONSTANT
=> "a root-level constant"
>> Object::MY_CONSTANT
=> "a root-level constant"
```

Note that there is not actually a “global scope” for constants. Root level constants are defined (and looked up) within the `Object` class. The expression `::MY_CONSTANT` is just a shorthand for `Object::MY_CONSTANT`.

**What if we want to know all the constants that are currently in scope?** We can use an instance method of `Module` class `Module#constants` to get that list. (In the file system analogy, this is like `ls` or `dir`)

There is also a class method of `Module` with the same name (confusingly). We can call `Module.constants` and that will returns all the top-level constants in the current program. This will include class names too. When I did `Module.constants` in `irb` it returned a couple dozen constants. 

One last useful method, before we wrap up! **What if we want to know the current path in the tree?** `Module.nesting` tells us that. (In the file system analogy, this is like typing `pwd` at your command prompt).

```ruby
>> Module.nesting
=> []
?> module M
?>   class C
?>     module M2
?>       Module.nesting
?>     end
?>   end
>> end
=> [M::C::M2, M::C, M]
```

`Module.nesting` answers the question where am I? It's an array that goes all the way up the hierarchy.

I hope the 5 minutes you spent reading about how constants work in Ruby, will save you from surprises in the future!

