---
layout: post
title:  "Upgrading Rails, Gemfile, Bundler, RubyGems"
---

One way to get free performance gains for your Rails application is by updating Rails and Ruby versions. In addition to features and security fixes, new releases of Ruby and Rails ship ongoing performance improvements to the language and framework.

I recently upgraded an app that I hadn't touched in two year. In this post, I'll share answers to some questions that often come up around RubyGems (`gem` command), Bundler (`bundle` command), and Gemfiles - ruby's infrastructure for sharing packages and managing dependencies.

Note: This one is beginner friendly. If you've been around Ruby for a while, you likely already know all this. But if you're a new dev or new to the Ruby/Rails ecosystem, this post will explain tools you've likely already encountered. 

First, let's talk about Bundler. One of the steps for upgrading Rails in an application is to run `bundle update`. What will this command do?

## Bundler, Gemfile, and Gemfile.lock
You've likely done `bundle install` many times when starting a new application or after adding a new gem to your `Gemfile`. So what is the difference between `bundle update` and `bundle install`? Also, what is that `Gemfile.lock` file that you see in your git changes?

To answer that, we need first talk about Bundler and how it works.

Bundler is the dependency management tool for Ruby. It tracks and installs the exact gems and versions needed to provide a consistent environment for our Ruby projects. The `Gemfile` is the place to list all of the gems needed for a project.

When you do `bundle install`, Bundler resolves versions and installs all of the gems listed in the `Gemfile`, as well as *their* dependencies. For example, you may only have 5 gems listed in your `Gemfile` but when you do `bundle install`, you see 26 gems being installed. 

Bundler remembers the exact versions of all of these gems and stores that in a special file `Gemfile.lock`. The `bundle install` command automatically updates `Gemfile.lock`. So the next time you run `bundle install`, Bundler can skip dependency resolution (i.e figuring out what version of each gem to install), and install the versions specified in `Gemfile.lock`. Moreover, `Gemfile.lock` is checked into version control so that cloning our project on another machine, and running `bundle install` will still install the same gem versions that were installed on our machine.

However, from time to time, you might want to update the gems you're using to the newest versions. Like when upgrading Rails, the instructions say to "change the Rails version number in the `Gemfile` and run `bundle update`". When we use `bundle update` in that case, Bundler will ignore the `Gemfile.lock` and resolve all of the dependencies again based on the constraints of the gems in the `Gemfile`. Updating the version of the `rails` gem will implicitly result in updating the version of its many dependencies like `activerecord` and `actioncable`. 

```ruby
# Gemfile.lock
rails (7.2.0.alpha)
      actioncable (= 7.2.0.alpha)
      actionmailbox (= 7.2.0.alpha)
      actionmailer (= 7.2.0.alpha)
      actionpack (= 7.2.0.alpha)
      actiontext (= 7.2.0.alpha)
      actionview (= 7.2.0.alpha)
      activejob (= 7.2.0.alpha)
      activemodel (= 7.2.0.alpha)
      activerecord (= 7.2.0.alpha)
      activestorage (= 7.2.0.alpha)
      activesupport (= 7.2.0.alpha)
      bundler (>= 1.15.0)
      railties (= 7.2.0.alpha)
```

Doing a `bundle update` can result in a significantly different set of gems, based on the requirements of new gems released since the last time you did an update. The `bundle update` command also updates the `Gemfile.lock` to reflect the versions it installed.

So far we've been talking about Bundler and the `bundle` command. But you've probably done something like `gem install devise`. So what is the difference between `bundle` and `gem` commands?

## RubyGems and `gem` command
You may have noticed the `source` line at the top of your `Gemfile`:

```ruby
source 'https://rubygems.org'

# Your Gems
```

The `gem` command comes from RubyGems. RubyGems is the default package manager for Ruby. It allows us to package Ruby libraries in a way that can easily shared and distributed. RubyGems handles installing, uninstalling, and managing Ruby gems on our local machine. It even provides commands for publishing gems to the RubyGems.org repository and searching for gems.

`Gemfiles` require a "source" entry, in the form of a URL for a RubyGems server. We can generate a Gemfile with the default rubygems.org source by running `bundle init`. 

While RubyGems and Bundler work together, they have different jobs. 

Bundler is typically used within the context of a specific project. You initialize Bundler for a project by creating a `Gemfile` that lists the project's dependencies. 

When you do `gem install <gem_name>`, RubyGems installs the gem globally on your system.

RubyGems is primarily responsible for managing individual gems and their versions, while Bundler is concerned with all the gems in a given project.

[RubyGems](https://rubygems.org/) and [Bundler](https://bundler.io/) are both open source tools that have been maintained by [Ruby Central](https://rubycentral.org/), which is an organization that has been supporting the Ruby and Rails community since 2001.

Lastly, here's how to create and publish your own gem for fun.

### Creating and Publishing a Gem

I created a "hello world" gem called `theleafnode` in 10 minutes. All it does is prints a message.

```ruby
gem install theleafnode

>> require 'theleafnode'
=> true
>> Theleafnode.greet
Hello from Bhumi. Read more at http://blog.theleafnode.com
```

You can create your own gem by using the following [instruction from bundler](https://bundler.io/guides/creating_gem.html) . I'll give you the tl;dr below:

1. Run `bundle gem theleafnode` and then `cd theleafnode`
2. Update the `theleafnode.gemspec` with metadata
3. Add your code, for what the gem is suppose to do, in `lib/theleafnoce.rb` and gem dependencies to `Gemfile`.
4. Build the gem `gem build theleafnode.gemspec`
5. Publish your gem `gem push theleafnode-0.1.0.gem` (will need to create an account on rubygems.org).

If I go and search https://rubygems.org/ for `theleafnode` gem, it's there!
