---
layout: post
title:  "Does using freeze prevent changing the value of ruby constants?"
---

One of my students recently asked about the `freeze` method in Ruby (as it came up in one of their interviews). So we popped into an `irb` session. And well...I was surprised. 

Aside: Something to keep in mind about learning. When you're surprised, when your predictions don't come true about how something works, it's a gift. An opportunity to uncover something, to understand deeper, etc. 

So let me share with you what I discovered! This is a quick but an interesting one. Here we go!

We know about constants in ruby. (Here's a refresher[add link to last post]). Constants in ruby are not really like constants in other programming languages. We can reassign them. But...we make them *real* constants by calling `freeze` and then we won't be able to change their values. Right...?

Not so much.

```ruby
>> MY_CONSTANT = "foo".freeze
=> "foo"
     
>> MY_CONSTANT = "bar"
(irb):47: warning: already initialized constant MY_CONSTANT
(irb):45: warning: previous definition of MY_CONSTANT was here             
=> "bar"                                                                   

>> MY_CONSTANT
=> "bar"
```

We can reassign the constant `MY_CONSTAN` to a new value even though we used `freeze`. 

So what is `freeze` good for then? If we can still reassign? It makes the value "foo" immutable.

```ruby
>> MY_CONSTANT << "bar"
(irb):46:in `<main>': can't modify frozen String: "foo" (FrozenError)
```

We cannot change the string "foo" by concatenating "bar" to it. We get a runtime error. We have freezed (frozen?) the *value* "foo". 

We get a runtime error if we try to modify the value "foo" *but* we can still reassign the *constant* to a brand new value (with a warning from the Ruby interpreter).

The important point to note is that there is a difference between changing the value of the object that the constant points to (this is protected by freeze) vs. *reassigning* the constant to point to some brand new value (freeze allows this).

The below code snippet illustrates this point. Using `object_id` is a good way to see that both `C1` and `C2` are pointing to the same object (string "hi"), which is frozen.

```ruby
>> C1 = "hi".freeze
=> "hi"
>> C2 = C1
=> "hi"
>> C1.object_id
=> 460
>> C2.object_id
=> 460

>> C2 << "bye"
(irb):5:in `<main>': can't modify frozen String: "hi" (FrozenError)

>> C2 += "bye"
(irb):6: warning: already initialized constant C2
(irb):2: warning: previous definition of C2 was here   
=> "hibye"

>> C2
=> "hibye"
>> C1
=> "hi"

>> C2.object_id
=> 480
>> C1.object_id
=> 460

>> C1.frozen?
=> true
>> C2.frozen?
=> false
```

Then we try to modify the value of "hi" with `<<` to concatenate "bye" to it. That gives us the expected runtime error "FrozenError". But we can reassign `C2` with `+=`, that creates a new string "hibye" and works fine. We verify that it's a new string with a different `object_id`. Now `C2` contains a value that is *not* frozen. And `C1` still contains the original frozen value, as we haven't reassigned that reference.

One related thing I am wondering about is the history of the **magic comment `# frozen_string_literal: true`** you may have seen in production ruby files and in rails code.  It freezes all of the strings in the file so that we don't have to litter our code with `.freeze` all over the place. There is some notion that this helps with performance as we are creating less garbage for GC (I'm not sure, will have to explore this in a later post).

I think there was talk of freezing all string literals by default in Ruby 3. That did not actually happen for backward compatibility reasons. (If you are more familiar with the history over the different versions of Ruby, feel free to reply and share)

One last question to explore, before we wrap up! **Is the behavior of `freeze` different if used with a variable vs. a constant?** 

Well, since what we are freezing is the value, it should not matter whether the reference is a variable or constant. Let's confirm.

```ruby
> foo = "hello".freeze
=> "hello"
> foo << "bye"
(irb):2:in `<main>': can't modify frozen String: "hello" (FrozenError)

> FOO = "hi".freeze
=> "hi"
> FOO << "bye"
(irb):4:in `<main>': can't modify frozen String: "hi" (FrozenError)

> FOO = "bye"
(irb):5: warning: already initialized constant FOO
(irb):3: warning: previous definition of FOO was here
=> "bye"

> foo = "bye"
=> "bye"
```

So we have variable `foo` and a constant `FOO`. We get a runtime error in both cases when we try to modify a frozen value with `<<`. For reassignment we get a warning for a constant. And reassigning a variable like `foo` is fine and natural thing to do. So all good there.

This confirms that freeze doesn't have anything to do with the *reference* (a variable or a constant) but with the *value* that the reference is pointing at. We can *freeze* the value but we can still *reassign* the reference to a brand new value. No problem. If the reference is a *constant* we'll get a warning is all.

Now `freeze` is much more clear to me, hope for you too.

P.S. Feel free to add to this if there is more about freeze I didn't cover. Happy to share replies and update this post.