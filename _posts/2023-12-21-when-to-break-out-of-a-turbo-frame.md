---
layout: post
title:  "How and when to break out of a Turbo Frame?"
---

I ran into this question earlier. Turns out there are multiple ways to break out of a Turbo Frame. 

But first, why would we need to break out of a Turbo Frame and how do Turbo Frames work? Let's get into it! This is a super short one.

Note: This post is beginner friendly, if you're new to Hotwire/Turbo or even new to Rails or programming. If you've been building with Hotwire for a while, there is likely nothing new in here for you. Starting with a quick overview to set the context, feel free to skip to the next section if you're familiar.

## Turbo Frame Overview

Turbo Frames are one of three features of Hotwire's `turbo-rails` library (Drive and Streams being the other two).

Note: when I write `Turbo` I mean the `turbo-rails` library. And I'll write `Frame(s)` as a shorthand for Turbo Frame(s).  

So what is the Big Idea behind Turbo Frames? The big idea is *scoped navigation*. Link clicks and form submissions, from within a Frame, trigger an AJAX request that is *scoped* to the Frame. 

All `<turbo_frame>` elements on a page must have a unique DOM `id`. A HTML response to a request that originates from within a `<turbo_frame>` is expected to have a `<turbo_frame>` element with a matching `id`. 

Turbo adds an HTTP request header "Turbo Frame" whose value is the `id` of the expected `<turbo_frame>`, to indicate this to the Rails controller action.

Once the above is satisfied, the code in the `turbo-rails` library takes care of the rest. i.e. it renders the response HTML within the right Frame element in the DOM. Without us writing any JavaScript. Simple and convenient.

The other Big Idea of Turbo Frames is its `src` attribute. I'll cover what we can do with `src` in a separate post. It's pretty interesting.

Side note: How do Frames compare to Streams? Turbo Frames are more specialized than Streams and target a single element in the response. Streams are more general purpose in that a `turbo_stream` response can target *any* element on the page given a DOM `id`. (more on Streams in a future post).

Now that we have a-lay-of-the-Turbo-land, let's answer our one specific question: How and when to break out of a Turbo Frame?

## Why break out of a Frame?

Imagine we have a list of books displayed on an `index` page. Each book is rendered inside its own Turbo Frame with a unique id (i.e. `book_1`). There's a button to add/remove a given book to/from a user's reading list within the book's Frame.

The button click will be scoped to the Turbo Frame. Which is what we want. We want to stay on the `index` page still and just update a 'reading list' counter somewhere on the page. So far so good.

Now, the book's title is a link (within the same Frame) and we want clicking that link to take us to that book's `show` page. With the code as is, that won't work.

The partial that renders each book into a Frame looks like this:

```html
<%= turbo_frame_tag dom_id(book) do %>
  <div class="book">
    <%= link_to image_tag(book.cover), book %>
    <div class="title">
      <%= link_to book.title, book %>
    </div>
    <div class="author">
      <%= book.author %>
    </div>
    <% if signed_in? %>
      <%= button_to "Add to Reading List", users_readings_path(book_id: book) %>
    <% end %>
  </div>
<% end %>
```

The book's controller `show` action looks like this:

```ruby
  def show
    @book = Book.find(params[:id])
  end
```

Which renders the `show.html.erb` template.

```html
<div class="header">
  <div>
    <h2 class="title">
      <%= @book.title %>
    </h2>
    <div class="author">
      <%= @book.author %>
    </div>
    <div class="description">
      <%= @book.description %>
    </div>
  </div>
  <%= image_tag @book.cover %>
</div>
```

Which does *not* have a `<turbo_frame id="book_1">`.

When we click the `book.title` link, Turbo will throw a `TurboFrameMissingError`. Since the controller `show` action renders the `show.html.erb` template, which of course does *not* have a `<turbo_frame>` element.

Hmm...what if we wrap the `show` template in a `<turbo_frame>` and give it a matching `id`? That *would* make the error go away. But the `show` page will be rendered within the `<turbo_frame>` on the books `index` page. 

That's silly and not what we want. So what's the solution to this issue?

We need to *break out* of the `<turbo_frame>`. We want the AJAX request to actually navigate to the `show` page and render the `show` template (and update the browser url and all that).

We can tell Turbo to do that by annotating our `show` link with a special attribute `data-turbo-frame`.

## Different ways to break out of a Turbo Frame

For the book's name linking to book's `show` page, it looks like this:

```html
<%= link_to book.title, book, data: {turbo_frame: "_top"} %>
```

which generate this HTML:

```html
<a data-turbo-frame="_top" href="/books/4">The Hobbit</a>
```

As we see above, **one way to break out** of a Turbo Frame is to add a the `data-turbo-frame` attribute to the element (links and forms) that we care about. 

It's for when we want most links/forms to operate within the Frame scope, but not all. We can annotate the exceptions with `data-turbo-frame` attribute (in `erb` it's `data: {turbo_frame: "_top"}`). 

**A second way to break out** of a Turbo Frame is to set the `target` attribute on the `turbo_frame` element itself like:

```html
<%= turbo_frame_tag 'my_frame', target: '_top' do %>
  <%= link_to 'Load reviews on a new page', reviews_path %>
<% end %>
```

The `target = "_top"` attribute makes it so that *any* link or form navigation from within that Frame drives the entire page instead of just the enclosing Frame. 

Both `data-turbo-frame` and `target` can also target *another* Turbo Frame on the page by setting their value to the `id` of that Frame (instead of the special value `"_top"`).

**The third way to break out** of a Turbo Frame is using `turbo-visit-control`. This is for when we want navigating to a given page to be treated as a new full-page reload, even if the request came from within a `<turbo_frame>`. 

One example of this is when clicking a link in a Frame redirects to a login page (due to an expired session or something). In this case, it’s better for Turbo to display that login page rather than treat it as an error. We can mark the login page with a meta tag like this:

```html
<meta name="turbo-visit-control" content="reload">
```

We can also use the `turbo_page_requires_reload` Rails helper. Turbo will perform a full page reload whenever Turbo navigates to a page with this meta tag, including when the request originates from a `<turbo_frame>`. 

This is what's recommended in the browser console when there is a `TurboFrameMissingError`.

> The response (200) did not contain the expected `<turbo-frame id="book_1">` and will be ignored. To perform a full page visit instead, set turbo-visit-control to reload.

These are the 3 ways to "break out" of a Turbo Frame that I came across. That wraps up this one!