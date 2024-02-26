---
layout: post
title:  "Building infinite scroll with Turbo Frames and Lazy Loading"
tags: hotwire
---

In this post, I continue the Hotwire series of building common UI patterns. In the book tracking demo app, we'll load more books from a collection of books as the user scrolls down a page, also know as infinite scroll. We also learn about pagination with the `pagy` gem.

Note: This implementation only uses Turbo Frames. There are other ways to build the same functionality with Turbo Stream and Stimulus with different tradeoffs. The goal here is to learn about the `loading` and `src` properties on Turbo Frames. We're simply using infinite scroll as an example use case to learn those things.

Note: Looks like Turbo 8 with morphing ~~is going to be~~ was released this week. I will dig into that in a later post. I look forward seeing how this use case is solved with morphing.

In the meantime, let's see one way to build infinite scroll using Turbo Frame's lazy loading feature. Before we add any `turbo-frame` elements though, we need to have pagination on our book's `index` view. We'll use the `pagy` gem for that.

## Configure Pagination

We can configure [`pagy`](https://github.com/ddnexus/pagy) with a few simple steps:

1. Add `include Pagy::Backend` to `application_controller.rb`. This allows us to call the `pagy` method from our controllers to get a paginated `@books` model.
2. Add `include Pagy::Frontend` to `application_helper.rb`. This allow us to call helpers like `@pagy.page` and `@pagy.next` to get the current and next page in our `index` view. There is also `@pagy_nav` that renders a page navigation bar.
3. We get the list of pages in our controllers with with `pagy` method like this:

```ruby
# books_controller.rb
def index
  @pagy, @books = pagy(Book.all, items: 5)
end
```

Now we're ready to introduce Turbo Frames.

## Lazy Loading using Turbo Frames

We need to learn about two `turbo-frame` properties, `src` and `loading`, in order to load follow-on pages as the user scrolls down.

The Turbo Frame `src` attribute is for setting the URL to be loaded, like `src: books_path`. When a `turbo-frame` element enters the DOM, an async `fetch` request will automatically be made to the `src` value. Changing the value of the `src` property (in the dev tools for example) will immediately navigate the `turbo-frame` element.

However, *if* that `turbo-frame` also has the `loading` attribute set to `"lazy"`, changes to the `src` value will **differ navigation until the element is visible in the viewport**. The `loading` attribute has two valid values: `"eager"` and `"lazy"`. The default value for `loading` is `"eager"`.

So, how do we take advantage of these `turbo-frame` attributes to build infinite scroll?

We can load the first 5 books in the book's `index` view. Then request the next page of books with `/books?page=2` in a `turbo-frame` with `id` of something like `"books_page_2"`. Then the next 5 books will be requested with a `src` set to `/books?page=3` and loaded inside a `<turbo-frame id="books_page_3">`. This feels recursive and seems complicated but the code is straightforward. 

To help visualize things, this is what the DOM tree will look like as we scroll:

 ![Screen Shot 2024-02-07 at 7.57.07 AM.png](https://assets.buttondown.email/images/464d3fc2-9d4a-48d7-963e-1e219c9bac3a.png?w=960&fit=max) 

Notice that the `turbo-frame` elements for the follow-on pages are nested inside the very first one. They all have `loading="lazy"` and a `src` and `id` attribute with a matching page number. 

Here's the entire `index.html.erb` for the book controller's `index` action that implements the infinite scrolling of books.

```ruby
<h1>Books</h1>

<%= button_to "Add Book", new_book_path %>

<div id="books">
  <%= turbo_frame_tag "books_page_#{@pagy.page}" do %>
    <%= render @books %>

    <% if @pagy.next %>
      <%= turbo_frame_tag "books_page_#{@pagy.next}",
                          src: books_path(page: @pagy.next),
                          loading: :lazy %>
    <% end %>
  <% end %>
</div>
```

The inner Turbo Frame is the one that fetches the next 5 books as the user scrolls down a page. For the value of the `src` attribute we use the handy `@pagy.next` helper. The `books_path(page: @pagy.next)` will result in the path `/books?page=N`. The `loading: :lazy` part ensures that no request is sent to the server if the user never scrolls down.

The outer Turbo Frame `turbo_frame_tag "books_page_#{@pagy.page}"` is there to wrap the `index` view response in a `<turbo-frame>` with an `id` that will match the `<turbo-frame>` that requested it. So the next set of books will be loaded inside the Turbo Frame that made the request.

That is all and it works. Couple more point before I wrap up:

As a mentioned in a [previous post](https://blog.theleafnode.com/build-search-with-hotwire/), the `src` attribute is powerful and comes in handy in a few use cases. The `loading` attribute is also useful outside of the infinite scroll use case. The original `turbo-frame` can be an empty placeholder, waiting for the lazy loaded content, if it ever becomes visible in the viewport.

With nesting Turbo Frames, things could become convoluted and difficult to keep track of though. So we should use it with care, and tradeoff the added complexity with user experience.
