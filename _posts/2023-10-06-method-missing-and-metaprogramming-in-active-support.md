---
layout: post
title:  "What is method_missing? Examples of metaprogramming in Rails ActiveSupport"
---

Have you ever wondered why we can write `Rails.env.production?` instead of having to check with equals like `Rails.env == "production"`?

We'll take a little tour of `ActiveSupport` code to answer this question about method calls and `method_missing`. Specifically the `StringInquirer` class, which is behind this little feature on `Rails.env`. 

On this tour, we'll also see other examples of metaprogramming concepts in Ruby. Such as `instance_variable_set` for dynamically setting instance variables, `class_eval` for executing a block in the context of an existing class, etc.

Let's get started!

Let's open up the [`StringInquirer` class](https://github.com/rails/rails/blob/832fb1de704899a230c83e7c966efac03a012137/activesupport/lib/active_support/string_inquirer.rb#L21). It's pretty short and there we see `method_missing` on line 27.

```ruby
class StringInquirer < String
    private
      def respond_to_missing?(method_name, include_private = false)
        method_name.end_with?("?") || super
      end

      def method_missing(method_name, *arguments)
        if method_name.end_with?("?")
          self == method_name[0..-2]
        else
          super
        end
      end
end
```

## What is method_missing?

Let's review how method calls work in Ruby. When we call a method on an object, Ruby looks for the method definition in the object's class. If it doesn't find it there, it goes up the class's ancestor chain, all the way to a class called `BasicObject`.

```ruby
>> String.ancestors
=> [String, Comparable, Object, PP::ObjectMixin, Kernel, BasicObject]
```

`BasicObject` is at the top of ruby's class hierarchy. It has an instance method called `method_missing`, that *all* objects in ruby inherit. So if ruby can't find the method we're calling anywhere else in the ancestor chain, it admits defeat by calling `method_missing`. 

This implies that we can override `method_missing` in a new class to 'catch' calls to methods that don't actually exist. These are sometimes called Ghost Methods. Why is this useful?

Let's play with `StringInquirer` to see.

```ruby
>> fruit = ActiveSupport::StringInquirer.new("apple")
=> "apple"
>> fruit.apple?
=> true
>> fruit.orange?
=> false
>> fruit.sldjlsd?
=> false
```

We can call arbitrary methods on `fruit` and it will return `true` for `apple` and `false` for everything else. Note that these are all predicate methods, ending in a `?`

That's all the `StringInquirer` class does.  But how does it reply correctly to arbitrary method names? 

## `StringInquirer` Explained

When we call a method `apple?` on `fruit` it eventually gets to `method_missing` of `StringInquirer`. In `method_missing`, it checks whether the name of the method ends with a `?`. If it does, we want to 'catch' it. If not, we pass it to `super`.

When we 'catch' the method, we return the value of this comparison `self == method_name[0..-2]`. The right hand side is just the method name without the `?`. The value of `self` is `fruit`. 

Remember we are in a method call. **The value of `self` during a method call is the *receiver*** (the object that the method was called on). In this case, `self` is an `ActiveSupport::StringInquirer` object, which is a subclass of `String`. (We will cover how the value of `self` changes in Ruby, in a later post) 

If the `method_name` does not end with a `?`, we don't want `method_missing` to catch that call. It passes to `super` which will throw 'NoMethodError' error. This is important because we don't want to catch all possible method calls.

```ruby
>> fruit.grow
/Users/bhumi/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/activesupport-7.0.0/lib/active_support/string_inquirer.rb:29:in `method_missing': undefined method `grow' for "apple":ActiveSupport::StringInquirer (NoMethodError)
```

We get our `NoMethodError` if we call `grow` on `fruit`.

One more thing, there is also this `respond_to_missing` in `StringInquirer`, we'll come back to that one.

But now we're ready to look at the [Rails sourcecode](https://github.com/rails/rails/blob/832fb1de704899a230c83e7c966efac03a012137/railties/lib/rails.rb#L72) that makes calls like `Rails.env.development?` possible.

```ruby
    def env
      @_env ||= ActiveSupport::EnvironmentInquirer.new(ENV["RAILS_ENV"].presence || ENV["RACK_ENV"].presence || "development")
    end

    # Sets the \Rails environment.
    #
    #   Rails.env = "staging" # => "staging"
    def env=(environment)
      @_env = ActiveSupport::EnvironmentInquirer.new(environment)
    end
```

Here are the methods for reading and writing `env`. It wraps the env name string in a class called `EnvironmentInquirer`. We haven't seen that class yet. But I bet that's a subclass of our `StringInquirer`. 

Yup [it sure is]([https://github.com/rails/rails/blob/main/activesupport/lib/active_support/environment_inquirer.rb#L7](https://github.com/rails/rails/blob/main/activesupport/lib/active_support/environment_inquirer.rb#L7)) `class EnvironmentInquirer < StringInquirer`.

The comment tells us that this class is doing some optimization for the three default environments, so it doesn't need to rely on the slower delegation through method_missing that StringInquirer uses. 

This class doesn't use `method_missing` directly but it has two nice examples of dynamic programming concepts for us to explore. `instance_variable_set` and `class_eval`. 

```ruby
    def initialize(env)
      raise(ArgumentError, "'local' is a reserved environment name") if env == "local"

      super(env)

      DEFAULT_ENVIRONMENTS.each do |default|
        instance_variable_set :"@#{default}", env == default
      end

      @local = in? LOCAL_ENVIRONMENTS
    end

    DEFAULT_ENVIRONMENTS.each do |env|
      class_eval <<~RUBY, __FILE__, __LINE__ + 1
        def #{env}?
          @#{env}
        end
      RUBY
    end
```

Here's what the code is doing:
- `instance_variable_set` will create 3 instance variables called @development, @test, @production. And set them to either true or false.
- Under the `class_eval`, we define 3 methods. `development?`, `test?`, `production?`. Those methods simply return the value of the instance variable with a matching name we set up earlier.

Both of these are examples of metaprogramming. We are dynamically setting instance variables and defining methods at runtime.

An aside: the `__FILE__, __LINE__ + 1` stuff in the heredoc makes it so if something goes wrong in this dynamically defined methods, the debugger can point us to a file name and line number in the code. I wasn't sure, I had to look it up, but that's what I think this stuff is for.

That's all. We've gotten to the bottom of this code exploration. We can see exactly how `Rails.env.production?` works now. It's a method that is dynamically defined on the `EnvironmentInquirer` class (which is a subclass of `StringInquirer` and has `method_missing` for all calls that end in a `?`). 

## What is `respond_to_missing`?

Before we wrap up, I said I'd come back to `respond_to_missing`. Why is that needed? 

Remember that Ruby has duck typing, and we can ask any object if it responds to a given method.

```ruby
>> fruit.respond_to?("apple?")
=> true
>> fruit.respond_to?("grow")
=> false
```

It's answering correctly because we are overriding `respond_to_missing` in `StringInquirer`. If we didn't, ruby will have no way of knowing that we're catching methods that end in `?`. With this line `method_name.end_with?("?") || super` in `respond_to_missing` we are telling ruby that we respond to all methods that end in `?`.

In general, overriding `respond_to_missing` whenever you override `method_missing` is a good thing to do, so our objects don't lie to us.

To wrap up, We confirm that `class` of `Rails.env` is `ActiveSupport::EnvironmentInquirer` which is a subclass of `ActiveSupport::StringInquirer` and that is why we can call those handy predicate methods.

```ruby
>> Rails.env.development?
=> true
>> Rails.env.class
=> ActiveSupport::EnvironmentInquirer
```

Beautiful. It's just a tiny little feature. But in this tiny feature of ActiveSupport, we saw many metaprogramming concepts in action. We used `method_missing` and `respond_to_missing`. We talked about method lookup and ancestor chain and the value of `self`. And we saw `instance_varialbe_set` and `class_eval`.

That's all I got. Hope you enjoyed this guided tour of some `ActiveSupport` code.
