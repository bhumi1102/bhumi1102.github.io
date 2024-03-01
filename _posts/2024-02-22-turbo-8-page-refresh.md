---
layout: post
title:  "When to use Turbo 8 Page Refresh"
---

Let's talk about Turbo 8 and the Page Refresh feature. Specifically *when* to use this feature in your application. 

I'm focusing on *when* to use Page Refresh. As *how* to enable Page Refresh is straightforward. The feature is very well explained in the official [announcement post](https://dev.37signals.com/a-happier-happy-path-in-turbo-with-morphing/) and [demo application](https://github.com/basecamp/turbo-8-morphing-demo). In fact, I will refer to the demo application with Projects and Tasks to illustrate when Page Refresh happens.

Note: Turbo 8 release also includes two other features, InstaClick and View Transition support. I'm not covering them in this post for the sake of length.

So, *when* do we use Page Refresh? What conditions need to exist in our user interface for Page Refresh to make sense for a given page/view?

The short answer is that Page Refresh is triggered when the URLs match. Meaning when a user submits a form and we redirect back to the same URL. Turbo will recognize that the paths match and instead of triggering a traditional Turbo Drive navigation, it will trigger a Page Refresh. 

So what does Page Refresh do? Both, Turbo Drive and Page Refresh, re-request the full page from the server. But while Turbo Drive would've replaced the whole `<body>`, Page Refresh will 'merge' the new content into the existing page, without touching the old content that didn't change. 

How does it do that? Using an algorithm implemented by [Idiomorph](https://github.com/bigskysoftware/idiomorph) for morphing one DOM tree to another. (aside: Idiomorph was developed to integrate with [htmx](https://buttondown.email/bhumi/archive/what-is-htmx/))

Why go through all the trouble of diffing the DOM and morphing? To preserving scroll position and other internal DOM state. 

Let's see this in-action with an example. I'll use the Projects and Tasks example from the official demo app. 

## Page Refresh With Form Submission
So how do we enable this Page Refresh feature? It requires very little code but we have to do *something*. 

Well, here are the steps:

1. Update your Gemfile to latest [turbo-rails](https://github.com/hotwired/turbo-rails/releases) library v2.0.x released in February 2024. This is the same as Turbo v8.0x. 
2. Add these `meta` tags to your `application.html.erb`

```html
# app/views/layouts/application.html.erb
<head>
  <meta name="turbo-refresh-method" content="morph">
  <meta name="turbo-refresh-scroll" content="preserve">
  ...
</head>
```

There is also a helper that adds both of these tags (although it didn't work for me): `<%= turbo_refreshes_with method: :morph, scroll: :preserve %>`

3. That's it. There is no step 3.

How do we check to see if it's working? We can submit a `form` that redirects back to the same path. 

In the demo application, we create a new task. A `task` `belongs_to` a `project`. The form for creating a new `task` is on its project's `show` page:

```ruby
<%# app/views/projects/show.html.erb %>
...

<%= render "projects/header", project: @project %>
<%= render "projects/add_task_form", project: @project %>
<%= render "projects/tasks", project: @project, tasks: @tasks %>
```

The form looks like this:

```ruby

<%= form_with model: [project, project.tasks.new] do |form| %>
  ...
<% end %>
```

and will be sent using `<form action="/projects/1/tasks" method="post">`.

When the `form` is submitted, the path of the page we are on is still the project's `show` page though (e.g. `/projects/1`). 

Since the `<form>` is POSTed to `/projects/1/tasks`, it's received by the  `TasksController#create` action, which looks like this:

```ruby
def create
  @task = @project.tasks.create! task_params

  redirect_to project_url(@project), notice: "Task was successfully created."
end
```

Notice that `redirect_to project_url(@project)`. Ah Ha. So, after we create a `task`, we redirect back to that Project's `show` page `projects/1`. This is what I mean by the paths matching. With Turbo 8 Page Refresh enabled, we will morph in the newly created `task` to its Project's `show` page. Two other elements, the progress bar and the project count, on the page will also get updated as a result of DOM morphing, without us doing anything explicitly (i.e. no Turbo Stream for each change).

In fact, **this is the pattern for when Page Refresh is used**. There is a page that we can think of as the "home base". When we interact with that page to create or update other "sub resources", we want to stay on the "home base" and have the updates be automatically reflected. While preserving scroll position and other state on the page. 

In the demo application, we can think of a Project's `show` page as the "home base". Tasks are the "sub resources" that we can create, update, and delete from the "home base". When we do, we'd like to stay on the "home page" and have the changes be merged into the DOM. So that `redirect_to project_url(@project)` is key when creating (as well as updating and deleting) a `task`. Typically, if we use scaffolding for example, when we create a new resource, we're redirect to *that* resource's `show` page or maybe that resources `index` page. But to trigger Page Refresh, we go back to the "home base".

Other examples of this "home base" page pattern (btw, I'm making up the terms "home base" and "sub resources". They are not official terms): We have a calendar and we want to stay on a given page but add/delete/update events. Or in a my book notes tracking application, if I were to add a concept of a "reading list". I would have a "home base" but add/delete/updates "books" to the "reading list", and update count and progress bar.

## Page Refresh When Data Changes on the Server
The other part of the Turbo 8 Page Refresh feature is about updating open browser windows when data changes on the server. This is done with `broadcasts_refreshes`.

```ruby
class Project < ApplicationRecord
  broadcasts_refreshes
end

class Task < ApplicationRecord
  belongs_to :project, touch: true
end

# /app/views/projects/show.html.erb
<%= turbo_stream_from @project %>
```

That's all there is to it, in terms of code changes to take advantage Page Refresh for when data changes on the server. 

Before we wrap up, one more thing. Is the feature called Turbo Page Refresh or Turbo Morph, what's the difference? Page Refresh is the new paradigm of automatically getting a partial page refresh (without defining Turbo Frames or Turbo Streams). Morphing and DOM merging are the technical details of how we achieve the refreshing (using the Idiomorph library) .

To summarize the answer to "When to use Page Refresh?":

Page Refresh is not used everywhere in an application. It's triggered 1. when data changes on the server if we have `broadcasts_refreshes` and 2. when a user submits a request to the server and the server redirects back to the same URL. Turbo will recognize that the paths match and instead of triggering a traditional Turbo Drive navigation, it will trigger a Page Refresh to morph in the changes to the DOM. The goal is to enable higher fidelity UI (i.e. preserve that precious scroll position).

One last thought, while the code changes to enable these features are minimal, there is behind-the-scenes complexity. In some use cases, the added fidelity of preserving the scroll position (and not jumping all over the page when you try to add a comment while reviewing a PR on Github for example) is worthwhile. But not in all cases, so we can *start* with Turbo Drive and add in more fidelity (and more complexity) if our context demands it.