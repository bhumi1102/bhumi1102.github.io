layout: post
title:  "Rails Console Commands Cheatsheet"
---

Rails console is a useful playground for trying out various view helper commands as well as experimenting with our ActiveRecord data model.

Here is a rundown of some things we can do in a Rails console session.

## Basic commands
### Reloading the Console
We can enter the Rails console by typing `rails console` or `rails c` in our terminal. When we do, our application files are cached for the duration of our session. If we make code changes, we can use the `reload!` command to update the console session. This is a shortcut and has the same effect as exiting the console and starting a new session.

### Searching command history and autocomplete
The unix shell commands also work in Rails console
- Searching commands with up/down arrows and also with `Ctrl+R` to search backwards through the command history.
- Autocomplete with `<tab>`  works on class name, method name or object/variable that are in scope in the console session. Hit `<tab>` multiple times to see all options that match. Useful if we can't remember the full method name.

```ruby
> User.w<tab><tab>

User.warn_on_records_fetched_greater_than User.with_options

User.warn_on_records_fetched_greater_than= User.with_reset_password_token

User.where User.writing_role

User.while_preventing_writes User.writing_role=
```

### Getting the value of the last expression with `_`

This one is handy. The underscore `_` always contains the value of the last expression that was evaluated by the console. 

Let's say we executed this command in the console

```ruby
> User.where("income >= ?", 1000000).order(:state)
```

We can do this to get at the results from the `_`.

```ruby
> users = _
> users.size
```

## View Helpers
ActionView helpers in Rails are convenience methods that allow us to format dates, string, numbers as well link to assets like images, videos and stylesheets. 

We can use the `helper` variable to test out these methods in our Rails console before embedding them in our view templates.

Here are some examples of cool things the view helpers can do:

```ruby
> helper.time_ago_in_words(65.minutes.from_now)
=> "about 1 hour"

> helper.time_ago_in_words(500.hours.from_now)
=> "21 days"

> helper.number_to_currency(1234567.89)
=> "$1,234,567.89"

> helper.number_to_human_size(1234)
=> "1.21 KB"

> helper.image_path("logo.svg")
=> "/assets/logo-a306f97bdfe7337812aaf2ca03415d755fc7ddc9d7b16e30b5f96a556cf6206f.svg"
```

## Route Helpers
We can try out route helpers in the console by calling the methods on the `app` object.

```ruby
> app.user_session_path
=> "/users/sign_in"

> app.user_registration_path
=> "/users"

> app.user_password_path
=> "/users/password"
```

If we make any changes to the `routes.rb` file, we need to call `reload!` to reload the console.

We can also issue requests like `app.get` and `app.post` on the `app` object, which calls the matching controller actions. The controller actions will return a `response` and we can inspect `response.body` and `response.header`

## ActiveRecord Commands

Another cool things about the Rails console is that we can play with our data models directly. We can create, update, and delete records and use any of the commands in the Active Record [query interface](https://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations) to query and find specific records.

For all the Active Record commands we can inspect the generated SQL and make sure it's what we expect. That's useful for testing out more complex queries using `.joins` and such before putting those in our application (to check for N + 1 issues for example)

```ruby
> Book.first
 
 Book Load (2.2ms) SELECT "books".* FROM "books" ORDER BY "books"."id" ASC LIMIT $1

> Book.count

 Book Count (5.5ms) SELECT COUNT(*) FROM "books"
```

Execusting Active Record commands like `update` and `destroy` will actually make changes to our database. We can start the Rails console in sandbox mode to experiment with data without the worry of accidentally messing up the database.

## Use sandbox to avoid changing database

Any changes we do in `rails console --sandbox` will be rolled back on exit. Changes like `User.destroy(1)`.

The Rails console starts in development environment by default but we can start it in other environments with `-e`. 

```ruby
$ rails console -e production --sandbox

Loading production environment in sandbox (Rails 7.0.0)

Any modifications you make will be rolled back on exit
```

The above starts Rails console in production environment in sandbox mode.

## Open source code for a given method

Let's say we are experimenting in the console and crafting some code for our application. We use a method of a class and now are curious to see exactly how it's implemented. We can open up the source code right from within the Rails console.

The below code opens my editor to the line number of the method `dollar_amount` of my `Plan` class.

```ruby
> location = Plan.instance_method(:dollar_amount).source_location

=> ["/Users/bhumi/Code/sloowreads/app/models/plan.rb", 60]

> `subl #{location[0]}:#{location[1]}`
```

Cool right?

