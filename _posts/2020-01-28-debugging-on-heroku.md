---
layout: post
title:  "Debugging on Heroku"
date:   2020-01-28
---

### Deploying, Migrating DB, Viewing Logs, Reading DB

**The errors and what I did to resolove then, in order of seeing them**

*Error* while doing Google login on https:

`ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  relation "identities" does not exist`
OH apparently DB tables have not been created yet…
Solvev by - did `heroku run rake db:migrate` and verified in heroku app dashboard that DB was created and had 9 tables

*Error* next one while trying to google login:

`ActiveRecord::StatementInvalid (PG::UndefinedColumn: ERROR:  column "2020-01-28" does not exist
2020-01-28T20:20:47.117922+00:00 app[web.1]: LINE 6:       AND dailies.date = "2020-01-28";`
Looks like the date format with double quotes is not liked by Postgres. I recall having to put it in for sqlite locally

Solved by: changing to single quotes (and verified by installing psql locally, connecting to heroku db by `heroku pg:psql` and testing it out, worked)

That's all the errors. sign-in with Google successful at https://healthspace.app I see the beautiful (to me) Dashboard with empty habit grid and 'add new habit' button and all that!

Also have done:
deploying (`git push heroku master`)
migrating db (`heroku run rake db:migrate`)
viewing logs (from broswer and also `heroku logs --tail`)
reading DB (`heroku pg:psql`)
---
Side today - clean up this warnings
```
-----> Detecting rails configuration

###### WARNING:

       You have not declared a Ruby version in your Gemfile.

       To set your Ruby version add this line to your Gemfile:

       ruby '2.5.5'

       # See https://devcenter.heroku.com/articles/ruby-versions for more information.

###### WARNING:

       There is a more recent Ruby version available for you to use:

       

       2.5.7

       

       The latest version will include security and bug fixes, we always recommend

       running the latest version of your minor release.

       

       Please upgrade your Ruby version.

       

       For all available Ruby versions see:

         https://devcenter.heroku.com/articles/ruby-support#supported-runtimes

###### WARNING:

       No Procfile detected, using the default web server.

       We recommend explicitly declaring how to boot your server process via a Procfile.

       https://devcenter.heroku.com/articles/ruby-default-web-server

-----> Discovering process types

       Procfile declares types     -> (none)

       Default types for buildpack -> console, rake, web

-----> Compressing...

       Done: 67.3M

-----> Launching...

       Released v15

       https://secret-harbor-24791.herokuapp.com/ deployed to Heroku
       ```