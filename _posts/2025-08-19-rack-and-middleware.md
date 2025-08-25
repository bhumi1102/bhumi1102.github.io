---
layout: post
title:  "How a web server passes an incoming HTTP request to your Rails application"
---

Have you ever wondered about the `config.ru` file in your Rails app? How does a web server (e.g. Puma) talk to your Rails app? What does it mean for Rails to be a Rack app? And what's a middleware? These are all related, let's understand how!

We know this much: when a client/browser sends an HTTP request to our Rails application, the router matches the path to a specific controller action, which processes the request and sends back a response. End of story, right? Well...so let's zoom in on the part *between* the incoming HTTP request and the Rails router. 

HTTP Request --> | ? | --> Rails Router --> Controller#Action

What's in that "box" with the question mark. There is a web server for one. Rails comes with a web server called Puma, which you've likely used. 

Puma (and Unicorn, Pitchfork, Falcon) is a Rack-compatible web server, which takes an incoming HTTP request, create a Ruby hash (called `env`), and then calls our Rails app. We won't go into how web servers work in this post (saving that topic for different post). But let's focus on how a web server *calls* our Rails app exactly and what's the `env` hash? 

That's where Rack comes in. 

### Rails Apps are Rack Apps

You've likely heard that before, what does it mean for a Rails app to be a Rack app? Rack is a very simple Ruby interface. All it means for a server to be Rack-based is that it implements the Rack interface - a method called `call(env)` that returns an array with `[status, headers, body]`. And that's it.

We can demonstrate Rack by building a simple Rack application and starting it with the `rackup` server. This file can be our entire app:

```ruby
# Put this in a file called config.ru
class HelloWorld
  def call(env)
    [
      200,                                # status
      { "content-type" => "text/plain" }, # headers
      ["Hello World!"]                   # body
    ]
  end
end

run HelloWorld.new
```

We can run it like this:

```bash
rackup config.ru
```

That starts the web server at port 9292. We can go to `localhost:9292` and see our "Hello World!". We're running a very simple Rack application!

In the case of Rails, The `Rails::Application` class (that all Rails apps extend) implements the `call(env)` method. So instead of a file with one method, we have an entire web application behind that `call(env)` call. 

More specifically, we can see the [`call(env)` in the engines.rb file](https://github.com/rails/rails/blob/4f595868f95c63849653446e5ab7a96f191974d3/railties/lib/rails/engine.rb#L532) as the `Application` class extends `Engine`. (A Rails app is also a Rails Engine - Engines will have to be a topic for another post too). 

Puma is what calls into the `Rails::Application`'s rack-based `call` method. And it does this with the `config.ru` file in our Rails app. Here is the entire `config.ru` file (the `ru` stands for rackup):

```ruby
# config.ru

# This file is used by Rack-based servers to start the application.

require_relative "config/environment"

run Rails.application
Rails.application.load_server
```

We can see the line `run Rails.application` in `config.ru`.

So what happens after that? Are we at the router yet? Almost! There are middlewares in the middle...

### What is a Middleware?
Middleware is software that sits _between_ an incoming HTTP request and the application. It processes requests *before* they reach the main app and/or responses *before* they’re sent back.

The Rails framework ships with a default middleware stack (you can see it in the [source code](https://github.com/rails/rails/blob/main/railties/lib/rails/application/default_middleware_stack.rb)). Rails uses middleware for logging, session management, serving static file, and such. There is also a Rails command to see a list of Rails middleware, in the exact order they are run:

```bash
$ bin/rails middleware
use ActionDispatch::HostAuthorization
use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::Executor
use ActionDispatch::ServerTiming
...
...
run YourRailsApp::Application.routes
```

And at the end of Rails's middleware stack is our application's routes! 

To dig into the call a bit more, the `app` object in the `app.call` below in the `call` method is not just our Rails app, it's the entire middleware stack.

```ruby
# railties/lib/rails/engine.rb

# Define the Rack API for this engine.
def call(env)
  req = build_request env
  app.call req.env
end

...

# The `app` object contains the middleware stack
def app
  @app || @app_build_lock.synchronize {
  @app ||= begin
    stack = default_middleware_stack
    config.middleware = build_middleware.merge_into(stack)
    config.middleware.build(endpoint)
  end
  }
end
```

Also, each piece of middleware is Rack-compatible and implements `call(env)` method, which can modify the `env` hash of the incoming request.

#### We Can Create and Replace Middleware
We can create our own custom middleware and we can also replace/remove a middleware that ships with Rails.

For example, we can add a class that implements the `call(env)` Rack interface to our application and configure it as middleware in `config/applicaiton.rb` with something like this:

```ruby
config.middleware.use MyCustomMiddleware
```

There are gems that add their own middleware in the Rails stack. For example, if you inspect your middleware for an app that has the Device gem installed, you'll see a `use Warden::Manager` line towards the bottom.

In addition to adding custom middleware, we can also replace or remove the middleware that Rails comes with. The reason for doing this are usually related to performance or security. For example, a Rails app in  `--api` mode removes many of the default middleware like `ActionDispatch::Cookies`, `ActionDispatch::Session::CookieStore`, or `Rack::Sendfile` that are not needed for a JSON API.

To wrap up, that concludes our dive into of how a web server calls our Rails application and ends up in the familiar router routing to a controller action.

### Try This At Home
I'll leave you with a little experiment you can try to demonstrate these concepts to yourself - figure out how to inspect the `env` Ruby hash that your app gets sent from Puma. 

One way to see the `env` hash is to create your own middleware that logs the hash (before the request is sent to the application controller. The other way is to log `request.env` from within a controller).

```ruby
class EnvLogger
  def initialize(app)
    @app = app
  end

  def call(env)
    Rails.logger.debug "=== Rack env ==="
    Rails.logger.debug env.inspect
    @app.call(env)
  end
end
```

and then add the middleware to `config/application.rb`:

```ruby
# Insert at the very start, before all other middleware
config.middleware.insert_before 0, EnvLogger
```

Now restart your Rails server and you'll see the `env` hash in your development log when you make a new request. It outputs a whole slew of key/values, here is the beginning:

```bash
=== Rack env ===

{"rack.version"=>[1, 6], "rack.errors"=>#<IO:<STDERR>>, "rack.multithread"=>true, "rack.multiprocess"=>false, "rack.run_once"=>false, "rack.url_scheme"=>"http", "SCRIPT_NAME"=>"", "QUERY_STRING"=>"", "SERVER_SOFTWARE"=>"puma 6.4.2 The Eagle of Durango", "GATEWAY_INTERFACE"=>"CGI/1.2", "REQUEST_METHOD"=>"GET", "REQUEST_PATH"=>"/books/298486374", "REQUEST_URI"=>"/books/298486374", "SERVER_PROTOCOL"=>"HTTP/1.1", ... }
```

In addition to the values that come from Puma, Rails also enhanced the `env` by adding values via its middleware stack (e.g. `action_dispatch.*` stuff).

With that, we've answered the questions we started with. We know about the `config.ru` file, about web servers calling it and the Rack interface and the `env` hash, and we've seen the Rail's middleware stack with our application's router at the end. That was fun. 
