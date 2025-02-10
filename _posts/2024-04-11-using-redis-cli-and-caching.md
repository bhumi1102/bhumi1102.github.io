---
layout: post
title:  "Using Redis CLI and Caching in Development"
---

This one is about Redis and playing with caching in development mode using the Redis CLI. 

Redis is everywhere. It's a simple key value store. In addition to caching, in a Rails environment, Redis is often used for background jobs, for Action Cable, for session store and more. We will only focus on the cache use case for examples in this post. 

This one falls in the category of get to know our tools better - `redis-cli` in this case. Let's first enable caching so we can play with the CLI.

## Caching in Development
We know caching is important in speeding up our pages. Caching is disabled in development mode by default though.

We can enable caching in development with `rails dev:cache`. This creates a file, `tmp/caching-dev.txt"`, whose presence is used to set the necessary config options. 

This is the relevant code in `developement.rb` that enables caching and sets `redis_cache_store` as the cache store:

```ruby
# Enable/disable caching. By default caching is disabled. Run rails dev:cache to toggle caching.
if Rails.root.join("tmp/caching-dev.txt").exist?
  config.action_controller.perform_caching = true
  config.action_controller.enable_fragment_cache_logging = true
  
  config.cache_store = :redis_cache_store
  config.public_file_server.headers = {
    "Cache-Control" => "public, max-age=#{2.days.to_i}"
}
```

So now what...can we see for ourselves stuff being cached? Yes, we can use the Redis CLI to peek at what's inside. 

After we enabled caching with `rails dev:cache` command, we may see entries similar to these in our logs:

```bash
Write fragment views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/1-20240325143547849277 (13.1ms)
Read fragment views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/2-20240325143620722062 (5.7ms)
```

One type of caching in Rails is fragment caching. When rendering a collection of partials, Rails can cache each partial fragment so it doesn't have to re-render it from scratch on future requests.

Let's find these cached fragments inside Redis.

## Redis CLI
Redis is a key value store. If we know the key, we can look up the value. Simple. The logs above give us a hint as to what keys are used to store these fragments in the cache store.

Aside: In the last few weeks, there's been some shake up and drama around Redis? Apparently. In life we shouldn't take things for granted, but I *have* taken Redis for granted. And I don't have time to follow what's going on...so I'll assume everything is fine. 

If you are running a Rails application, you likely have Redis installed and a server running. But in case not, here's an example of how to do it on a mac: 

```bash
# starts the server in the foreground
$ redis-server

# OR
# To start the server in the background,
# and restart each time you login.
$ brew services start redis

# To check to see if a server is running
$ brew services info redis
redis (homebrew.mxcl.redis)
Running: ✔
Loaded: ✔
Schedulable: ✘
User: bhumi
PID: 493
```

Now, we can connect to the `redis-cli`. Here's how to create key value pairs:

```bash
$ redis-cli
127.0.0.1:6379> SET "name" "bhumi"
OK
127.0.0.1:6379> GET "name"
"bhumi"
```

We don't want to create new keys though. We want to find the relevant keys for the message partial fragments. Here's how to do that:

```bash
127.0.0.1:6379> KEYS views/messages/*
1) "views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/5"
2) "views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/1"
3) "views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/4"
4) "views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/3"
5) "views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/2"
```

We have 5 fragments cached. We can see the value of one of those fragments:

```
127.0.0.1:6379> GET "views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/2"
"\x01x\x9c\xed\\\xcb\x8e\xe3\xc6\x15\r\x82\xc00\xfc\a\xd9\x11\x1a ;\xbaER\x12\xa5\xf1L'\xa2\xdeo\xb5D\xbdh\x04r\x91,>$\x92E\x91E\xbd\x96\xde\x19Fl\xc4\xf0\xc2\x9b \x9bx\xe3/\xc8\xf7\xe4\a\x92OHQ\xa2\xd4\x92Z\xdd\x9e\xeeit\x80\xb46\xd3R\xd5\xad[\xf7q\xea\x14\x9b\xf7\xf6\xfc\xee\xd3/?\x
..."
```

The value is bunch of hex bytes so I can't read it, maybe you can. But there it is!

In general, to browse keys in Redis we use patterns with wildcards such as `KEYS user/*`. 

Another way to look for keys is to use the `SCAN` command, which is more suitable in production environments. It returns a cursor and it can be executed against a busy server without impacting other operations: `SCAN 1 MATCH views/*`

You can learn about how to use [`SCAN`](https://redis.io/docs/latest/commands/scan/) and other commands in the [Redis CLI docs](https://redis.io/docs/latest/develop/connect/cli/). For example, we can do `redis-cli --bigkeys` to get a bunch of information about the size of keys:

```bash
$ redis-cli --bigkeys
...
Biggest string found '"views/messages/_message:a67c226048b769af2f280cc2e7acf30a/messages/2"' has 3062 bytes
...
```

Most of the time, we can rely on the higher level abstraction of caching, Active Job, etc. and not need to look at keys inside Redis. But it's nice to know that you *can* always look under-the-hood and see what's what for yourself.