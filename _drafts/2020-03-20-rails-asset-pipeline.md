---
layout: post
title:  "Rails asset pipeline"
date:   2020-03-20
---

Notes from this Heroku [article](https://devcenter.heroku.com/articles/rails-asset-pipeline) 

There are two ways you can use the asset pipeline on Heroku: 1. Compiling assets locally. 2. Compiling assets during slug compilation (which just heroku's compiling step when deploying I think?)


This compiles the assets locally
`RAILS_ENV=production bundle exec rake assets:precompile`

Generates a public/assets/manifest-md5.json file. If you check in this file, Heroku will detect and then not attempt compiles assets (I think)


--- tips, ideas ---
"When I make an app with rails as the backend now I use create-react-app or vue-cli and have separate repos for the front end and back end." - I may need to do something like this as front-end in rails seems not fun so far.


--- Facts ---
Yarn = bundler for javascript. The creator of bundler, yahuda katz, was involved in it's develoment ('compititor' to npm which is also a package/dependency manager)

setting `remote = 'true'` property on a <form> makes an ajax request

Pattern = Backend Rails API, Front end JS MVC (2011 backbone, today vue.js?)

Manifest Files (application.js and application.css) - Once you’ve placed your assets in their logical locations, you can use manifest files to tell Rails (via the Sprockets gem) how to combine them to form single files. So sprockets reads applicaiton.* files and follows the directives in there (the //= require stuff). So the order of those require statements matters I think. Hm.

Sprokets = Ruby library for compiling and serving web assets. It features **declarative dependency management** for JavaScript and CSS assets, as well as a powerful preprocessor pipeline that allows you to write assets in languages like CoffeeScript, Sass and SCSS.

Warning about directives in application.js (from sprokets readme) - "Note: Directives are only processed if they come before any application code. Once you have a line that does not include a comment or whitespace then Sprockets will stop looking for directives. If you use a directive outside of the "header" of the document it will not do anything, and won't raise any errors."

Asset pipeline has the default **load path** of app/assets/javascript, lib/asset/javascript, vendor/asset/javascript and you *could* configure other dirs to be on the load path in appliction.rb or something.

Asset Pipeline in development: You can turn off debug mode by updating config/environments/development.rb to include: config.assets.debug = false
When debug mode is off, Sprockets concatenates and runs the necessary preprocessors on all files. With debug mode turned off the manifest above would generate instead:

config.serve_static_assets configures Rails itself to serve static assets. Defaults to true, but in the production environment is turned off as the server software (e.g. Nginx or Apache) used to run the application should serve static assets instead. Unlike the default setting set this to true when running (absolutely not recommended!) or testing your app in production mode using WEBrick. Otherwise you won´t be able use page caching and requests for files that exist regularly under the public directory will anyway hit your Rails app

--- Questions to dig into ---
How are these files used? - app/asset/config/manifest.js and config/initializers/assets.rb

jquery-pjax what was that all about (mentioned in 2011 railsconf talk).

Relationship of jquery with Rails over the years? I was made the defualt framework in Rails 3, it was removed with Rails 5 or 6. I manually added the gem `jquery-rails` in my habit tracker project. Also where does jquery_ujs come into play?

Why is the heroku deployment complaning about yarn executable not found?
`Yarn executable was not detected in the system.`
This SO indicates that that may be a problem
`https://stackoverflow.com/questions/49419598/pushing-to-heroku-error-yarn-executable-was-not-detected-in-the-system`

Rails 5 by default using webpacker? how does asset pipeline relate to webpacker? Do I need one and not the other?

Do I need to have nodejs installed locally? and yarn?

What is the Ruby buildpack and why is node used? This snippet might have a clue:
"If you were previously using therubyracer or therubyracer-heroku, these gems are no longer required and strongly discouraged as these gems use a very large amount of memory.

A version of Node is installed by the Ruby buildpack that will be used to compile your assets."

config.serve_static_assets - understand this better, I should not have this turned on in prod unless I am testing with webbrick it sounds like. Otherwise my web server (nginx or puma?) should serve static assets. rails itself shouldn't be serving static assets.


---- Some answers ----
Good answer from [reddit](https://www.reddit.com/r/rails/comments/9zg7fe/confused_about_the_difference_between_sprockets/) - webpack, webpacker, asset pipeline, sprokets, yarn, npm
`The current state of things is really confusing. Basically, you can replace sprockets/the asset pipeline with a webpack. A single webpack replaces a single asset manifest, but unlike sprockets, it packages JS, CSS, and images together.
Why use Webpack? You get ES6 support out of the box (thanks to Webpacker, which is the library that configures Webpack for your Rails environment and abstracts some concepts and bridges the gap between Rails and Webpack). That’s a nice plus. You get modern JS things like tree shaking, modularity, etc. Basically, if you want to develop modern JS in a Rails app, you can use Webpacker.
npm: this manages JS dependencies (think of it as bundler and rubygems, but for JS). Rails didn’t really have a good solution for this before. You had to use a Rails gem that wrapped the library or rails-assets.
Yarn: this also manages JS dependencies. Using npm (lol). You can find resources that go into greater detail why one might use yarn instead of npm to manage JS dependencies. The Rails way is to use Yarn.
So let’s say you want to install a JS dependency via Yarn. How do you use that in your application? With either sprockets OR a Webpack! Both work. You can include your JS library installed via Yarn in a sprockets manifest or import it into a file in your Webpack. Your call.
Also, you don’t have to choose. You can use the asset pipeline and Webpacks. I have a project that has this somewhat standalone video player built into it. That player is built using Vue.js and packages its JS/CSS with Webpack (which is really cool because we can use .vue components). The rest of the app uses the asset pipeline/sprockets.
I’m by no means an expert here. I’ve been a rails developer for 8 years and this topic still confuses me. If I’ve made any mistakes or left anything out, I hope someone can drop in and correct me. This is what I’ve gathered from trying so sort this out myself.`

