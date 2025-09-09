---
layout: post
title:  "Lexxy Action Text Editor, Rendering Markdown in Rails (Rails 8.1.0)"
---

Rails 8.1 beta was announced at Rails World 2025 in Amsterdam last week. New things in this release include Active Job continuations, structured event reporting, local CI, deprecated associations. And more to come like Turbo offline, Active Native Push, Active Record tenanted, etc.

We are going to explore the fun Markdown stuff in Rails 8.1. 

I have a book notes app using Action Text with the new Lexxy rich text editor (instead of Trix). Let's explore Lexxy and how Rails' controllers can render Markdown now - as a first-class citizen, for AI friendly use cases. 

## Lexxy Hello World

I added Lexxy to an existing app, but let's make a brand new app for the sake of this demonstration. Following instruction in the Lexxy [README](https://github.com/basecamp/lexxy), here are the steps:

1. Create a new app, `rails new hello_lexxy` and `cd hello_lexxy`
2. Add the beta gem to your `Gemfile`, `gem 'lexxy', '~> 0.1.4.beta'`, and `bundle install`
3. Install Action Text and run migrations, `bin/rails action_text:install`. This will use the Trix editor by default but we will override that in a bit.

Next, we need to configure Lexxy by adding its JS and CSS to our app. This is a bit manual for now, I'm sure there'll be an installer in the future:

1. Using important maps for `lexxy.js`:

```ruby
# importmap.rb
pin "lexxy", to: "lexxy.js"
pin "@rails/activestorage", to: "activestorage.esm.js" # to support attachments
```
and 
```JavaScript
// app/javascript/application.js
import "lexxy"
import * as ActiveStorage from "@rails/activestorage"
ActiveStorage.start()
```

2. Add `lexxy.css` to our `appliction.html.erb` directly:

```HTML
<%= stylesheet_link_tag "lexxy" %>
```

3. The last step is to edit this Action Text file, `app/views/layouts/action_text/contents/_content.html.erb`, so that Action Text will render the Lexxy editor for `has_rich_text` fields (instead of Trix):

```ruby
<div class="lexxy-content">
  <%= yield -%>
</div>
```

We're almost ready to see Lexxy in action, as soon as we scaffold a `post` model:

```bash
$ bin/rails generate scaffold post title:string
```

and update the model, controller, and view to add a rich text attribute called `body`:

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  has_rich_text :body
end

# app/controllers/posts_controller.rb
class PostsController < ApplicationController
...
  private
    def post_params
      params.expect(post: [ :title, :body ])
    end
end

# app/views/posts/_form.html.erb
<%= form_with(model: post) do |form| %>
  ...
  <div>
    <%= form.label :body, style: "display: block" %>
    <%= form.rich_text_area :body %>
  </div>
  ...
<% end %>
```

We can see the new and shiny Lexxy editor when we render the form to create/update a post. We can see the the `lexxy-editor` element in the HTML. 

A feature I wanted for a while is the live preview when typing Markdown and the auto-formatting when pasting Markdown. Cool to see.

Lexxy has many other features and improvement over Trix. For example, we can disable attachments by passing `attachments: false` (per the README, though it didn't quite work for me). It will be nice to not have to edit `trix.css` with `display: none`.

Some other exciting Lexxy features:

- Syntax highlighting for code and the nifty way to create a link by pasting an URL on top of the desired text.
- Support for configurable prompts like `@` mention or bringing up commands with `/` or `#`, etc.
- Previewing attachments like PDFs and Videos.

In addition to introducing Lexxy for Action Text, Rails 8.1.0 beta also makes it easy to respond to Markdown requests and render Markdown directly.

## Rendering Markdown from Controllers
The [PR for Markdown rendering](https://github.com/rails/rails/pull/55511/files) is a short one. It registers a `MIME::Type` for Markdown, `text/markdown`, and adds a new `:markdown` renderer using the existing mechanism in Rails (which I coincidently covered in a [previous post](https://theleafnode.com/inertia-and-rails/) about Inertia): 

```ruby
add :markdown do |md, options|
  self.content_type = :md if media_type.nil?
  md.respond_to?(:to_markdown) ? md.to_markdown : md
end
```

The intent of this feature is use it for rendering Markdown stored in your database. But we can also demonstrate `render :markdown` by simply rendering a raw Markdown string from a controller:

```ruby
def about
  content = <<~MD
    # Hello from Rails 8.1.0
  
    We **love** Markdown!
  MD
  
  render markdown: content
end
```

When we navigate to `/about`, we can see the raw Markdown content and the `text/markdown` Content Type header as well. We can also do `/about.md` or `/about.markdown` since those MIME types are registered in Rails.

Note that this feature doesn't have anything to do with converting Markdown to HTML, it renders *Markdown* directly. Treating `.md` as a first-class response format makes it really handy to implement AI integrations with Rails. For example, AI tools can request `/posts/123.md` and get raw Markdown without needing extra parsing.

Apparently, we can have `*.md.erb` files as well with templated markdown with dynamic Ruby content injection. Pretty cool right? Hope you have fun playing with Markdown in Rails.


P.S. I wonder if that's the most number of times the word "Markdown" is used in one page?