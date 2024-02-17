---
layout: post
title:  "How to build responsive search with Turbo Frames and Stimulus"
---

We're building a simple responsive search using Turbo Frames and Stimulus. 

Specifically using `data-turbo-frame` on a `<form>` element to update a search results section on the page. 

We also add search-as-you-type with a little bit of JavaScript in a Stimulus controller.

This post is beginner friendly. If you're new to Hotwire, I focus on being thorough and don't skip over anything.

Sidenote: Using Turbo Frames is *one* way to build this search. With Hotwire there're usually multiple evolving ways to implement something. We can also build search with a Turbo Stream response (When Hotwire was first released, it wan't possible to respond with Turbo Streams to `GET` requests. That's not the case anymore. This [github issue](https://github.com/hotwired/turbo/issues/463) has some history if you're curious). And of course, the upcoming Turbo 8 morphing will simplify things and minimize code changes needed for common use cases. Regardless, it's worth understanding how to build search with Turbo Frames and learn more about the `src` attribute. 

Here's a screenshot of what this search looks like (I was going to add a gif but that didn't work within email. It's on my twitter profile):

 ![Screen Shot 2024-01-18 at 1.16.24 PM.png](https://assets.buttondown.email/images/5144cbf9-9028-4f24-bb3a-f7c510eb662c.png?w=960&fit=max) 

Let's get into how to build this responsive search:

So if we do nothing, when we submit a search form, the default experience with Rails 7 is that Turbo Drive will swap out the entire `html` body with the response from the server. It does *not* do a full page reload. Which is good *but* the search input field will still loose focus. And that's not so nice for the user. 

We can progressively enhance this behavior by introducing a Turbo Frame around the *search results* area on the page. This will allow the search input to retain focus, so that the user can keep typing.

Note that we'll only put the *search results* into a `turbo-frame` element. We can't put the *search form* inside that `turbo-frame` as well because the entire `turbo-frame` will get updated with the response and the input field will *still* loose focus.

Let me show you what I mean:

## Responsive Search with Turbo Frames

In this example, we're going to search a list books by their titles. The actual search is simply an `ActiveRecord` query. Not worrying about doing anything fancy there. 

The interesting part is the search form, that's at the top of our books's `index.html.erb` view.

We modify the default `index` view to add search with these 3 steps:

1. Wrap the search results in a `turbo-frame` element and give it a unique `id`, as in `<%= turbo_frame_tag "search_results" do %>` below.
2. Add a search form and target that `turbo-frame` element *from* the search `form` element by adding `data-turbo-frame = "search_results"` attribute to the `form` element.
3. To update the url each time a search form is submitted, add the `data-turbo-action = "advance"` attribute to the `form` element as well. This tells the browser to append the new url to the browser's history. This improves the UX as it allows a user to bookmark a search url and use the browser back button. 

Here's what the `index.html.erb` view looks like with those changes:

```ruby
<div class="search">
  <%= form_with(url: books_path, method: :get,
                data: { turbo_frame: "search_results", turbo_action: "advance" }) do |f| %>
    <div>
      <%= f.search_field :title, placeholder: 'Title...',
            value: params[:title] %>
    </div>
    ...
    <div>
      <%= f.submit "Find Books" %>
    </div>
  <% end %>
</div>

<%= turbo_frame_tag "search_results" do %>
  <div class="books">
    <%= render @books %>
  </div>
<% end %>
```

The controller can simply have an `index` action that does the search using user submitted `params` and renders the results using the default `index.html.erb` view. Or selects *all* books if there are no search params.

```ruby
  def index
    @books = Book.search(params)
  end
```

The `search` method on the `Book` model simply does an Active Record query on the book's title, something like `Book.where("lower(title) LIKE ?", "%#{title.downcase}%")`. We won't focus on that part as it's not relevant.

With these code changes in place, let's see how the search form with this `data-turbo-frame` attribute actually works.

### How targeting a `turbo-frame` from a `form` works

When a `form` element that *targets* a `turbo-frame` with `data-turbo-frame` attribute is submitted, it actually doesn't submit the request to the server. 

What it does is that it adds a `src` attribute to the target `turbo-frame`, with the value equal to the search `url`. In this case something like 

```html
<turbo-frame id="search_results" src="http://localhost:3000/books?title=beyond" complete="">
```

And when a `turbo-frame` has a `src` attribute, it automatically requests the specified url. We can see this in the dev tools network tab.

An `http` request header named `Turbo-Frame` with the value of `search_results` is also added to that request.

The `index` action of the Books controller will handle that request. The `index` view, that's rendered in response to the search form submission, will indeed contain a `turbo-frame` element with a matching value as in `<turbo-frame id="search_results">`. 

The content of that `turbo-frame` in the response are the books that match the search criteria. They'll be extracted from the response and used to update just the search results part of the page, the `<turbo-frame id="search_results">` element. (The rest of the `html` in the response will be ignored).

Hence completing this dance.

All this while not touching the search form and therefore keeping the focus on the search input. Which is the better UX that we were after. 

We can leave things here and that's better than the default.

But the next obvious enhancement is to update the search results as the user types, without having to hit enter or click a button.

## Search-as-you-type with Stimulus

In order to allow users to search as they type, we need to submit the search form automatically. We can use a Stimulus controller to do this. (I go over stimulus in a previous post, in case you'd like a refresher it's [here](https://buttondown.email/bhumi/archive/what-is-stimulusjs/))

We create a Stimulus controller called `search_form_controller.js` in `app/javascript/controllers` with the command `rails g stimulus search-form`.

This controller is attached to our `<form>` element and has an action called `submit` that submits the form on user `input`.

```ruby
<%= form_with(url: books_path, method: :get, data: { turbo_frame: "search_results", turbo_action: "advance", controller: "search-form", action: "input->search-form#submit" }) do |form| %>
```

The two new attributes we're attaching is `data-controller` and `data-action`.

The controller looks like this:

```JavaScript
import { Controller } from "@hotwired/stimulus"
import debounce from "debounce";

export default class extends Controller {
  initialize() {
    this.submit = debounce(this.submit.bind(this), 350)
  }

  submit() {
    this.element.requestSubmit();
  }
}
```

In addition to the `submit` method, this stimulus controller also has a little debounce functionality using a [library](https://www.npmjs.com/package/debounce#debouncefn-wait-options) to avoid sending excessive network requests as the user types. Instead of calling `submit` for every `input` event, it'll wait for 350ms (in this case) since the last request. 

And that's all that's needed to build our responsive search functionality.

### Turbo Frame `src` Attribute

It's worth calling out that the entire functionality of our search hinges on adding this `src` attribute with the desired `url` to the `search_results` Turbo Frame. 

The Turbo Frame `src` attribute is really powerful. We can use it to lazy load `turbo-frames` for other use cases as well.

Updating the `src` attribute even works if we change its value from the dev tools. Try it out, it's cool.

One last thing before we wrap up: when we add a Turbo Frame or a Turbo Stream, it does add a bit of complexity to our application. In this case, the trade-off is worthwhile to allow a more responsive search experience for the user. 

