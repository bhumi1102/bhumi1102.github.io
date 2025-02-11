layout: post
title:  "RailsUJS vs. Mrujs vs. Request.js"
---

Rails 7 officially removed including [rails-ujs](https://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html#replacements-for-rails-ujs-functionality) by default. So does that mean we should stop using rails-ujs then? And what are Mrujs and Request.js?

In this quick post, I answer those questions.

Let's start with a quick summary of what rails-ujs all about:

- rails-ujs is a rewrite of the jquery-ujs gem without the jquery dependency. (ujs = unobtrusive javascript. Basically keep js out of the markup and add 'data' attributes to the html to add js behavior)

- rails-ujs was [moved into Rails itself](https://github.com/rails/rails/commit/ad3a47759e67a411f3534309cdd704f12f6930a7) in Rails 5.1.0.

- Also rails-ujs is written in CoffeeScript apparently - we can read [the code here](https://github.com/rails/rails/tree/main/actionview/app/assets/javascripts/rails-ujs)

- rails-ujs allows us to do the following things:
    - make non-GET requests from `<a>` links (e.g. `data-method="delete"`)
    - make forms, links and buttons submit data asynchronously with Ajax
    - add confirmation dialogs for actions
    - disable submit button after form submission

rails-ujs also takes care of setting the CSRF token as a hidden input field inside forms. This article goes into [more detail with rails-ujs examples](https://www.ombulabs.com/blog/learning/javascript/behind-the-scenes-rails-ujs.html)

Okay but **since rails-ujs is not included with Rails 7, what are we suppose to do? We're suppose to use Turbo instead.** This basically implies instead of `data-method="delete"` we need to add `data-turbo-metho="delete"`. In our ERB templates it looks like this:
`<%= link_to "Delete post", post_path(post), data: { turbo_method: "delete" } %>`

That seems like a simple enough change from using rails-ujs to Turbo. So what is mrujs for then? 

Well, the change to using Turbo is minor for *new* projects but imagine migrating an existing project with hundreds of links with `data-method` and such from rails-ujs to Rails 7 Turbo. **It would be kind of a pain. Mrujs was created to solve that.** According to the [mrujs site](https://mrujs.com/) it strives to be a drop-in replacement for rails-ujs. Meaning we can keep doing `data-method` and all that. So mrujs is a successor to rails-ujs that is maintained. (Learn more about [how mrujs compares](https://mrujs.com/references/comparison-to-ujs))

What about [request.js](https://github.com/rails/requestjs-rails) then? Mrujs is for replacing all of UJS. **If you're only using AJAX from rails-ujs, swtiching to request.js may make sense.** With request.js, an AJAX call in a stimulus controller goes from this
```ruby
Rails.ajax({
      url: this.data.get("url").replace(":id", id),
      type: 'PATCH',
      data: data
    })
```

to...this (after adding the npm package)

```ruby
import { patch } from '@rails/request.js'
...
patch(this.data.get("url").replace(":id", id), {body: data})
```

Here is more detail about [how to use request.js](https://github.com/rails/request.js#how-to-use)

So in summary, we should move to Turbo for UJS functionality in new Rails 7 applications. For existing applications or while migrating we can keep using rails-ujs or we could switch to the more modern mrujs. And if only need the AJAX part of rails-ujs, we can use request.js.
