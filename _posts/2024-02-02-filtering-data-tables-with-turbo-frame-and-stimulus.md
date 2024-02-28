---
layout: post
title:  "Filtering Data Tables with Turbo Frame and Stimulus"
---

This one is about Hotwire. We'll add filtering to some books using Turbo Frames and Stimulus. 

Note: If you've been building with Hotwire for a while, there is likely nothing new here for you. If you haven't yet tried Hotwire, this will be a useful read and help you form a mental model of the core concepts. This post is closely related to a previous post where I implemented [responsive search](https://blog.theleafnode.com/build-search-with-hotwire/). And lastly, yes Turbo 8 will simplify certain things, but it's still worth understanding how to use Turbo Frames this way.

Here's a screenshot of the application with filtering:

![filtering books](/assets/images/filtering_data_tables_with_turbo_frame_and_stimulus_screenshot.png) 

We'll use a reading/book notes tracking application. Let's filter a list books a user has read by genre and something I call 'reading mode'.

Sidenote: 'reading mode' is something I made up that I personally use when I track books I've read. I used to be a 'finisher' and felt I had to finish any book I start. I've longed changed my perspective on that. Most books are longer than they need to be or simply not worth my time at a given point in my life. So now I consume books in different 'modes' and track *how* I read them and when/why I stopped reading. My 'reading_mode' values are 'read', 'reread', 'skipped', 'parsed'. 'skipped' is when I read 20% of the book to give it a fair shot and then say 'no thank you'. 'parse' is when I go through the table of content and pick and choose what I read. Most programming books fall in the 'parsed' mode. Although last year I did read *Metaprogramming Ruby 2* front to back and that was worthwhile. Books like *On Writing* and *On Writing Well* are marked 'reread' . Anyway, this is turning into a whole post but that'd be for a *different* newsletter. Let me get back to building filtering with Hotwire.

Back to filtering books. If we did nothing, how would filtering work? Well, selecting a 'genre' or 'reading_mode' in the form will cause a request to be submitted to the server. In a Rails 7 application, Turbo Drive will swap out the `<body>` from the response. 

So no full page reload but we can do *even* better using Turbo Frames. Ideally we want to leave the `<form>` alone and only update the part of the page that has the filtered results. Here's how we do that:

## Data Table Filtering with Turbo Frame and Stimulus

We modify the default Book Controller's `index` view to add filtering with the following steps:

1. Add the filters dropdown at the top of the `index` page in a `form`. 
2. Wrap the filtered books results in a `turbo-frame` element and give it a unique `id`, as in `<%= turbo_frame_tag "results" %>` below.
3. Target that `turbo-frame` element *from* the filter `form` element by adding `data-turbo-frame = "results"` attribute to the `form` element.
4. To update the browser url each time the form is submitted, add the `data-turbo-action = "advance"` attribute to the `form` element as well. This tells the browser to append the new url to the browser's history and allows the user to bookmark a url and use the browser back button.
5. Add a Stimulus controller with `data-controller` to auto submit the form on `input` event using `requestSubmit()`. To allow the user to filter/search without having to click a button. 

Here's what the `index.html.erb` view looks like with those changes:

```ruby
<%= form_with(url: books_path, method: :get,
        data: {
          turbo_frame: "search_results",
          turbo_action: "advance",
          controller: "search-form",
          action: "input->search-form#submit"
        }) do |form| %>
  <div class="filters">
    <div>
      <%= form.select :genre, @genres,
        include_blank: "Any Genre", selected: params[:genre] %>
    </div>
    <div>
      <%= form.select :reading_mode, @reading_modes,
        include_blank: "Any Mode", selected: params[:reading_mode] %>
    </div>
  </div>
<% end %>

<%= turbo_frame_tag "search_results" do %>
  <p><%= @books.count %> books found: </p>
  <div class="books">
    <table>
      <thead>
        <tr>
          <th>Title</th>
          <th>Author</th>
          <th>Genre</th>
          <th>Format</th>
          <th>Reading Mode</th>
          <th>Date Read</th>
        </tr>
      </thead>
      <tbody>
        <%= render partial: 'book', collection: @books %>
      </tbody>
    </table>
  </div>
<% end %>
```

(I'm reusing the Stimulus controller from building search hence the `search-form` and `search_results`. I should've called the Stimulus controller `form` and the turbo-frame `results`.)

The actual filtering happens as an `ActiveRecord` query. Not doing anything fancy there, as that's relevant for this Turbo Frame demonstration. Just something like `Book.with_genre(params[:genre]).with_reading_mode(params[:reading_mode])` in a `filter` method on the model, which the controller calls.

```ruby
def index
  @genres = Book.pluck(:genre).uniq
  @reading_modes = Book.pluck(:reading_mode).uniq

  @books = Book.filter(params)  
end
```

The `form` makes a `GET` request to the `BooksController#index` action. The `index` action returns the same `index` template above, with the `@books` instance variable set to the filtered list of books as specified by `params`. 

The response has a `turbo-frame` tag with the matching `id` from the request, so the results are replaced into the `turbo-frame` element and nothing else on the page changes (the response also has HTML for the `form` which is ignored, as it's outside the `turbo-frame` element.)

## A *Pattern* is Beginning to Emerge

If you remember the [previous post on building search](https://blog.theleafnode.com/build-search-with-hotwire/) with Hotwire, you may notice that overall implementing filtering is *very* similar to building search. In fact, it's exactly the same.

**The core pattern for building searching or filtering** is this: we can target a 'results' `turbo-frame` from a `form` element by adding `data-turbo-frame` to the `form`. When the `form` is submitted, instead of making a request to the server, it adds a `src` attribute to the target `turbo-frame`. The `turbo-frame` then fires a request and the response updates just the `turbo-frame` with the results, leaving the `form` and rest of the page alone.

**Another parallel pattern is about using Stimulus** to auto submit the form so that the user doesn't have to click a button. We get a more snappy experience with searching/filtering/sorting. Note that I used the same Stimulus controller on the filter form as the search form. Stimulus controllers are reusable.

That is all there is to it.

Of course we can filter by other fields, add sorting by all of the fields. We could add a range slider to the 'date_read' field. We can incorporate gems like `pg_search` or `meilisearch` to query the results. And we can add pagination to the results. 

I resisted the urge to include all that in this post. My goal is to make the reader (and me) feel like they really 'get' the key concepts (and *not* feel bogged down with extraneous details). We can tweak and build more features on top of that understanding (I may cover sorting in a later post).

