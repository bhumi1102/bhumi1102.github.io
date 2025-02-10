---
layout: post
title:  "Rails Performance: How to Use rack-mini-profiler Tool"
---

This one is about performance and how to measure things in a Rails application with the `rack-mini-profiler` tool.

Note: This post is going to be one of many in a Rails Performance series. Future posts will cover different types of caching in Rails, Active Record, database indexes, lessons from my experience doing ongoing performance work for a large health tech app with hundreds of thousands of user, and much more. 

Let's start with Profiling (If I was writing a book, this might not be the first lesson but gotta start somewhere!).

So what is profiling? Profiling is about analyzing an application's behavior to see how much time is being spent in different parts (and how much memory is being used). One goal of profiling is to pinpoint specific areas that are slow, and therefore are a bottleneck in the total request time (or consuming excessive resources and leading to high server bills).

Now, let's see how to use `rack-mini-profiler` for profiling a Rails application. If you've never seen `rack-mini-profiler`, I included a bunch of screenshots below to help you visualize its features.

## Profiling with `rack-mini-profiler`

The [`rack-mini-profile`](https://github.com/MiniProfiler/rack-mini-profiler)  is a tool (rack middleware) that displays a speed badge for every HTML page, along with (optional) flamegraphs and memory profiles. It's designed to work both in production and in development.

This is what the speed badge looks like:

![image1]()

Notice that in addition to showing how many milliseconds it takes to render each of the templates, we also see "query time". We can actually click on the "sql" links to see more details.

The query details screen shows the actual SQL queries that were run and also the line numbers in our code files that triggered those queries. This information can be very useful when pinpointing what's slow on a page.

![image2]()

That all looks like useful information. So how do you set this up in your Rails application?

## Installing `rack-mini-profiler`
All you have to do to start using this tool is add the gem(s) to your `Gemfile` and `bundle install`:

```ruby
gem 'rack-mini-profiler'

# Optional. For memory profiling
gem 'memory_profiler'

# Optional. For call-stack profiling flamegraphs
gem 'stackprof'
```

The `rack_mini_profiler` gem will identify database gems (e.g. `pg` or  `mysql`), if they are loaded already and insert instrumentation for SQL queries. In other words, order matters so add `rack_mini_profiler` *below* the `pg` and `mysql` gems in your `Gemfile`.

It's possible to use `rack-mini-profiler` in production. This requires some [more configuration](https://github.com/MiniProfiler/rack-mini-profiler?tab=readme-ov-file#access-control-in-non-development-environments) and storing the results in a data store like Redis.

## Flamegraphs and Memory Profiles
Once we add the `stackprof` gem to the gemfile, we can add the `?pp=flamegraph` to the url to see the Flamegraph. A Flamegraph is a visual of the call stack over time and can illustrate which functions or code paths are responsible for consuming the most CPU time.

Here is an example of a Flamegraph:

![image3]()

For profiling memory, we can install the `memory_profiler` gem. Here is an example of the information we get once we add `?pp=profile-memory` to the url:

![image4]()

That's a lot of information (and we're not getting into how to make sense of it in this post). Let's use the tool to profile some code with Collection caching.

## Profiling Collection Caching
We will use `rack-mini-profiler` to look at times with and without the `cache: true` in the `render` call below, for fun.

```ruby
<%= render partial: 'messages/message', collection: @messages, cached: true %>
```

What is Collection caching? Rails can cache each partial fragment and serve future requests more efficiently by retrieving the cached version of each partial rather than re-rendering from scratch. This can be useful when you have a large number of items in a collection and rendering each one individually has a cost that adds up to be significant. 

The `cached: true` tells the `render` method that the individual partial templates in the collection are cached. Rails will fetch all cached templates at once instead of one by one. The templates that haven't yet been cached will be written to cache and can be multi-fetched on the next render. 

Multi-fetching the templates from the cache store, such as Redis, is supposed to be faster than fetching them one by one. Could we see this with `rack-mini-profiler`?

Here is page load with `cache: true`:

![image5]()

Here is the page load without Collection caching:

![image6]()

This *seems* to show that adding `cache: true` helps. 

This is a very cursory look. We will dig into caching and how to test caching in development with `rails dev:cache` and Redis CLI in a future post.
