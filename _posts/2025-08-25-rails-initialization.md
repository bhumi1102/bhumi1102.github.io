
---
layout: post
title:  "Rails Initialization: What happens when you run Rails Server command"
---

What happens when we run the `rails server` command? A lot. It initializes the entire Rails framework, boots the application, and starts the web server. In this post, we go over the timeline of what happens and why it's useful to understand this in practice. 

Here's a sketch of the timeline:

```bash
$ bin/rails server
   ↓
bin/rails script → railties/lib/rails/commands/server/server_command.rb
   ↓
Load config.ru
   ↓
require config/environment.rb
   ↓
require config/application.rb
   ↓
Rails.application.initialize!
   ↓
Framework + app code loaded
   ↓
Puma (web server) starts listening
   ↓
Incoming HTTP request
   ↓
Puma builds Rack env, calls Rails.application.call(env)
   ↓
Middleware stack → Router → Controller → Response
   ↓
Response back to Puma → client
```

In a [previous post](https://theleafnode.com/rack-and-middleware) we focused on the bottom 4 things in the sequence above. In this post, we'll zoom in on the middle part -  `environment.rb` --> `applicaiton.rb` --> and all the things that happen when `Rails.application.initalize!` is called. (The `rails server` command also wraps the Puma web server, we won't cover Puma much in this post).

Understanding the Rails initialization process will allow us to know:
- When different parts of the framework are available to use. (e.g.  `Rails.logger`, `Rails.env`, `Rails.root`, etc.).
- The order in which things are run so that we can debug sequencing issues (e.g. An initializer having a dependency on another initializer).
- How to customize and "hook into" the initialization process with load hooks and initialization hooks.

### The `environment.rb` File as the Entry Point
The `config/environment.rb` file is the starting point of the Rails initialization process. You can boot the Rails framework by adding `require config/environment.rb`  in a file (this is what `config.ru` does). The `environment.rb` file is generated in all new Rails applications but you rarely need to open it:

```ruby
# config/environment.rb

# Load the Rails application.
require_relative "application"

# Initialize the Rails application.
Rails.application.initialize!
```

The first line requires `config/application.rb`, this is the familiar file that defines our application that we often modify:

```ruby
# config/application.rb

require_relative "boot"

require "rails/all"

Bundler.require(*Rails.groups)

module MyAmazingApp
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 8.0
  ...
  end
end
```

Loading `application.rb` loads all the Gems in our application's `Gemfile`. With the `require "rails/all"` line, it loads all Railties and Engines that register initializers for all framework components such as Active Record, Active Job, etc.

Then, subclassing our `Application` class from [`Rails::Application`](https://github.com/rails/rails/blob/main/railties/lib/rails/application.rb) causes an `inherited` callback to run, which sets up `Rails.root` among other helpers. Therefore `Rails.root` is available in the body of the `MyAmazingApp::Application` class onwards, but not before the subclassing. 

The second line of the `environment.rb` file above calls the `initialize!` method, which kicks off setting up the entire Rails framework and booting everything. Let's see that in more detail.

### `Rails.application.initalize!`

When `initialize!` is called, it runs all registered initializers in order, including the ones from Railties mentioned above. The `initialize!` method is defined in the `Rails::Application` class:

```ruby
# railties/lib/rails/application.rb

# Initialize the application passing the given group.
def initialize!(group = :default) # :nodoc:
  raise "Application has been already initialized." if @initialized
  run_initializers(group, self)
  @initialized = true
  self
end

...

def initializers # :nodoc:
  Bootstrap.initializers_for(self) +
  railties_initializers(super) +
  Finisher.initializers_for(self)
end

# railties/lib/rails/initializable.rb
def run_initializers(group = :default, *args)
  initializers.tsort_each do |initializer|
  initializer.run(*args) if initializer.belongs_to?(group)
  end
end
```

All groups of initializers, `Bootstrap`, Railtie, and `Finisher`, are run in topological order (`tsort` comes from Ruby and it respects dependencies among tasks with `:before` and `:after` rules).

The `Bootstrap` initializers prepare the application
(like initializing `Rails.logger`).

The custom initializers in our application under `config/initializers` are also run as part of the `initialize!` call. They are registered via [`load_config_initializer`](https://github.com/rails/rails/blob/36bd50c82b46046c9f352e06fa552221586028d8/railties/lib/rails/engine.rb#L643) in `Rails::Engine` Railtie. They are run in alphabetical order by file names in that directory.

The `Finisher` initializers set up the middleware stack and the router and are run last.

At this point, the Rails initialization process is complete and the application is booted and ready. Now the web server can be started to accept incoming requests.

### Try This
Here is a little experiment you can try to demonstrate initialization hooks and when framework components are available:

This is an example of an `config.after_routes_loaded` initialization hook. Add this to your `config/application.rb`:

```ruby
module MyAwesomeApp
  class Application < Rails::Application
    # Fires very early
    config.before_initialize do
      puts "[before_initialize] Rails.application.routes.routes.any? #{Rails.application.routes.routes.any?}"
    end

    # Fires after frameworks & initializers, but BEFORE routes are loaded
    config.after_initialize do
      puts "[after_initialize] Rails.application.routes.routes.any? #{Rails.application.routes.routes.any?}"
    end

    # Fires AFTER routes are fully loaded from config/routes.rb
    config.after_routes_loaded do
      puts "[after_routes_loaded] Rails.application.routes.routes.any? #{Rails.application.routes.routes.any?}"
      puts "[after_routes_loaded] Number of routes: #{Rails.application.routes.routes.size}"
    end
    ...
end
```

Then start the Rails server and you will see that routes are not loaded until later in the initialization process:

```ruby
$ bin/rails server
=> Booting Puma
=> Rails 7.1.3 application starting in development 
=> Run `bin/rails server --help` for more startup options
[before_initialize] Rails.application.routes.routes.any? false
[after_initialize] Rails.application.routes.routes.any? false
[after_routes_loaded] Rails.application.routes.routes.any? true
[after_routes_loaded] Number of routes: 45
```

We'll wrap it up here. You can follow how a Rails application accepts incoming HTTP requests once a web server is up and running in the [previous post](https://theleafnode.com/rack-and-middleware/).
