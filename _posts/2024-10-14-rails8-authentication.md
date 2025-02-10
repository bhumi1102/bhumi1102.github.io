---
layout: post
title:  "Rails 8 Authentication"
---

I was at Rails World in Toronto couple weeks ago and Rails 8 beta was announced with the goal of simplifying deployment with Kamal 2 and Thruster, reducing the number of accessory services by using SQLite with Solid Cache, Solid Cable, and Solid Queue, new simplified asset pipeline called Propshaft (which replaces Sprokets) and Authentication. We are going to talk about Authentication in this post.

Rails ships with authentication now? So what does that mean? Is this something that'll replace the `devise` gem or free your application from having to build a custom authentication solution. Not exactly. It's a starting point. Here's how it works.

### Rails 8 Authentication Generator
Rails 8 introduces a generator that adds authentication related models, controllers, views, routes, and migrations directly into your application. It uses the `bcrypt` gem for passwords. 

To use this feature in a new application, you run `rails generate authentication`. Here are all of file it modifies and new files the generator adds:

```ruby
$ rails generate authentication
      invoke  erb
      create    app/views/passwords/new.html.erb
      create    app/views/passwords/edit.html.erb
      create    app/views/sessions/new.html.erb
      create  app/models/session.rb
      create  app/models/user.rb
      create  app/models/current.rb
      create  app/controllers/sessions_controller.rb
      create  app/controllers/concerns/authentication.rb
      create  app/controllers/passwords_controller.rb
      create  app/mailers/passwords_mailer.rb
      create  app/views/passwords_mailer/reset.html.erb
      create  app/views/passwords_mailer/reset.text.erb
      create  test/mailers/previews/passwords_mailer_preview.rb
        gsub  app/controllers/application_controller.rb
       route  resources :passwords, param: :token
       route  resource :session
        gsub  Gemfile
      bundle  install --quiet
    generate  migration CreateUsers email_address:string!:uniq password_digest:string! --force
       rails  generate migration CreateUsers email_address:string!:uniq password_digest:string! --force 
      invoke  active_record
      create    db/migrate/20241010215312_create_users.rb
    generate  migration CreateSessions user:references ip_address:string user_agent:string --force
       rails  generate migration CreateSessions user:references ip_address:string user_agent:string --force 
      invoke  active_record
      create    db/migrate/20241010215314_create_sessions.rb
```

Next step is `rails db:migrate`, since migrations are created for `user` and `session` tables.

Then, if you check `routes.rb` and go to `session/new`, you'll see a form that accepts an email and a password with "sign in" button. This form routes to the `SessionsController` which was added by the generator. The core functionality around session management is the `Authentication` Controller Concern, which is included in `ApplicationController`.

There is also a "forgot password?" link on the "sign in" page that navigates to the `passwords/new` path. The `PasswordsController` runs through the flow for sending a password reset email. The mailers for this are also set up by the generator at `app/mailers/password_mailer.rb` and renders the following email to send to the user:

```ruby
# app/views/passwords_mailer/reset.html.erb
<p>
  You can reset your password within the next 15 minutes on
  <%= link_to "this password reset page", edit_password_url(@user.password_reset_token) %>.
</p>
```

After running the Authentication generator, you do need to implement your own *sign up flow* and add the necessary views, routes, and controller actions. There is no code generated that creates new `user` records and allows users to "Sign up" in the first place. This is something we'd need to wire up in our applications.

Here is a list of modified files:

```ruby
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  modified:   Gemfile
  modified:   Gemfile.lock
  modified:   app/controllers/application_controller.rb
  modified:   config/routes.rb

Untracked files:
  (use "git add <file>..." to include in what will be committed)
  app/controllers/concerns/authentication.rb
  app/controllers/passwords_controller.rb
  app/controllers/sessions_controller.rb
  app/mailers/passwords_mailer.rb
  app/models/current.rb
  app/models/session.rb
  app/models/user.rb
  app/views/passwords/
  app/views/passwords_mailer/
  app/views/sessions/
  db/migrate/
  db/schema.rb
  test/mailers/previews/
```

So that's what included as part of the Authentication feature with Rails 8. In a future post, I will go over the key authentication related code in the generated files.

One of the reasons given for why Rails did not ship with authentication as a core feature up until now was that authentication falls in the "business logic" domain and often requires bespoke custom solutions for real world applications. This was certainly true at my previous job. 

We had a healthcare application that employees of a company who had purchased it from our company could log into (B2B2C I guess). Not anyone could just sign up for an account and login. We would get data about who can access our app in these files called "eligibility files" from several hundred employers daily. This eligibility data needed to be updated in realtime in production (e.g. if an employee left the company or if they added a new dependent) to allow only the appropriate users to be able to register for an account and use the application. Registration itself was an elaborate process to collect and verify employee details as the data presented in the application was unique to each user based on their health plan, deductible, etc. status. Anyway, I spent a lot of time on authentication, accounts, and registration. There was inherent complexity given the domain.

With Rails 8, `rails generate authentication` will take care of the basics of creating a user model to store email + encrypted pwd and verify against those credentials to allow users to sign in. It will also facilitate password recovery and send password reset emails. The generated code uses `has_secure_password` and `authenticate_by`.
