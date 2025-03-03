---
layout: post
title:  "Using Tailwind CSS with Rails"
---

Let's talk about doing CSS with Rails. Rails asset management has evolved quite a bit over the years. The aim has been to *simply* but in the process we are left with multiple choices for doing something. And confusion about which way to use when, what's the "old way" vs. "new way". 

I found this to be true for using Tailwind CSS with Rails. In this post, I focus on some specific questions around installing and using Tailwind:

- Should I install Tailwind with the `tailwindcss-rails` gem? or some other way? (And what is the `tailwindcss-ruby` gem anyway?)
- What is the `cssbundling-rails` gem?
- What happens when I do `rails new my_app --css tailwind`?
- What is the difference between using Tailwind with or without Node in Rails?
- What is PostCSS? And Tailwind CLI? (And what's changing in Tailwind v4 about using it with Rails?)

There appear to be two ways to use Tailwind with Rails, without Node.js and with Node.js (no-build or build I guess).

### Tailwind CSS without Node.js

If you are creating a brand new Rails projects, then you can do `rails my_app --css tailwind`. This will

- add the `tailwindcss-rails` gem to the `Gemfile`, 
- do `bundle install`, and 
- run `./bin/rails tailwindcss:install`. 

For an existing Rails projects, you add the gem and then run the same `tailwindcss:install` command manually. So what does this install command do?

Aside: I noticed a gem called `tailwindcss-ruby` as well and wondered what it was for. It simply wraps a Tailwind CLI executable (which I talk about below). It's installed as a dependency of `tailwindcss-rails` gem.

The following files are created or modified when we run `tailwindcss:install` (with Tailwind v3, it's a little different with v4):

```bash
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  modified:   .gitignore
  modified:   Gemfile
  modified:   Gemfile.lock
  modified:   app/assets/config/manifest.js
  modified:   app/views/layouts/application.html.erb

Untracked files:
  (use "git add <file>..." to include in what will be committed)
  Procfile.dev
  app/assets/builds/
  app/assets/stylesheets/application.tailwind.css
  bin/dev
  config/tailwind.config.js
```

The purpose of all of the changes above is to build Tailwind's CSS file and add it to the Rails application layout (`app/views/layouts/applicaiton.html.erb`) so that we can use Tailwind utility classes in our application.

There is a `tailwindcss:build` task, which builds the `tailwind.css` file and places in in `app/assets/builds`. The install command adds the following line to `application.html.erb` to include this file:

`<%= stylesheet_link_tag "tailwind", "inter-font", "data-turbo-track": "reload" %>`

Since this setup is Node free, does it use Import Maps then? Not exactly. It uses the Rails Asset Pipeline to compile tailwind directly.  

To use Tailwind, in development, we need to run `tailwindcss:watch` task as a separate process along with `rails server`. The install command generates a `Procfile.dev` so we can run both processes using `foreman`:

```
# Procfile.dev
web: bin/rails server
css: bin/rails tailwindcss:watch
```

In production, `tailwindcss:build` task is automatically attached to `assets:precompile`, so before the asset pipeline digests files, the Tailwind output will be generated.

The install command also adds a Tailwind config file `tailwind.config.js` (in v3. It's different in v4, which I talk about below).

So that answers some of the questions above. Next, let's see what this `cssbundling-rails` gem is about.

### Tailwind CSS with Node.js

Chances are you already have a build step for your application, in which case you can use Tailwind with Node.

We can use the `cssbundling-rails` gem for that (good explanation in the [README](https://github.com/rails/cssbundling-rails)). To install Tailwind, we run:

```
./bin/bundle add cssbundling-rails 
./bin/rails css:install:tailwind
```

That install command above will add a `package.json` file to install tailwind related Node packages via `yarn`. (`autoprefixer`, `postcss`, and `tailwindcss` in tailwind v3. In v4, the first 2 packages are not needed).

It will also add a `build:css` script to `package.json`, which looks like this:

```json
"scripts": {
    "build:css": "tailwindcss --postcss -i ./app/assets/stylesheets/application.tailwind.css -o ./app/assets/builds/application.css",
    ...
    }
```

And the installer will update `Procfile.dev` to use the above build script `css: yarn build:css --watch`.

Here are the files created or modified by `./bin/rails css:install:tailwind`:

```
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  modified:   .gitignore
  modified:   Gemfile
  modified:   Gemfile.lock
  deleted:    app/assets/stylesheets/application.css
  modified:   app/views/layouts/application.html.erb
  modified:   bin/dev

Untracked files:
  (use "git add <file>..." to include in what will be committed)
  Procfile.dev
  app/assets/builds/
  app/assets/stylesheets/application.tailwind.css
  package.json
  yarn.lock
```

So I alluded to Tailwind v4 and how things are a little different between v3 and v4. Tailwind v4 was released in January 2025. At a high level, it gets rid of PostCSS and the JavaScript config file. 

Which begs the questions: What do we use instead of PostCSS? and where do we put Tailwind configuration if not in the `tailwind.config.js` file?

### Using Tailwind CLI (instead of PostCSS)
So what is Tailwind CLI? Tailwind CSS was originally written in JavaScript and distributed as an `npm` package, which means you've  had to have `Node` and `npm` installed to use it. Tailwind CLI was announced in December of 2021 to solve this problem:

*"Today we're announcing a new [standalone CLI build](https://github.com/tailwindlabs/tailwindcss/releases/latest) that gives you the full power of Tailwind CLI in a self-contained executable — no Node.js or npm required. You get all the power of our standard npm-distributed CLI in a convenient, portable package — no dependencies required."*

A little history: back in 2021, the Tailwind team had a vision of writing Tailwind in Rust, so that it can be distributed without a JavaScript runtime like `Node` and without requiring JS ecosystem tooling like `npm` or `yarn`. They did that for TailwindCSS v4 released January 2025.

Using Tailwind CLI, my build script in v4 looks like this:

```
"scripts": {
    "build:css": "npx @tailwindcss/cli -i ./app/assets/stylesheets/application.tailwind.css -o ./app/assets/builds/application.css --minify"
  }
```

### Where to add configuration in Tailwind v4 without the `tailwind.config.js` file?
The reason Tailwind v4 is getting rid of the JavaScript config file, I am inferring, is that the JS file would need a JS runtime (i.e. Node.js) to process it.

Instead, the recommendation is to add customizations directly in the CSS file where you import Tailwind. Such as `app/assets/stylesheets/application.tailwind.css`:

```CSS
@import "tailwindcss";

@theme {
  --font-display: "Satoshi", "sans-serif";  

  --breakpoint-3xl: 1920px;

  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
  --ease-snappy: cubic-bezier(0.2, 0, 0, 1);
}

@plugin "@tailwindcss/forms";
```

The release notes state that "the new CSS-first configuration lets you do just about everything you could do in your `tailwind.config.js` file, including configuring your design tokens, defining custom utilities and variants, and more."

That covers all my questions. Hope that clarifies what's what for using Tailwind with Rails!
