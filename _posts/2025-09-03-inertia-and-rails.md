
---
layout: post
title:  "How Inertia works under the hood with Rails and React"
---

Let's explore how Inertia works under the hood. Because many production codebases use Rails *with* some frontend thing (which is not Hotwire), in practice. From Rails API + single page app (SPA) to Rails views along with React components in some hybrid setup, with varying degree of complexity. Inertia is an option in this space and it *seems* simple. What allows it to serve as a glue between Rails and JavaScript components. How does it work?

## What is Inertia?
Inertia a tool that allows you to create client-side rendered, single-page apps, *without* turning our Rails app into an API only mode. It has adapters for several server-rendered frameworks (e.g. Rails) and several Javascript component libraries (e.g. React) that you can mix and match. 

In an Inertia app, Rails controllers return data directly to React components. There is no JSON API, no client-side routing, or client-side state management libraries.

Before we go into *how* it works, let's build a simple "hello world" Rails app with Inertia and React so we have something to play with.

### Inertia Hello World

To get a Rails + Inertia app going:

1. Create a new Rails app, `rails new hello_inertia`
2. Add the `inertia-rails` gem, `bundle add inertia_rails` 
3. Run the installer, `bin/rails generate inertia:install`

The Inertia installer adds `vite-rails` gem as the frontend build tool. Here are all the files added/modified by the Inertia installer:

```bash
  modified:   .gitignore
  modified:   Gemfile
  modified:   Gemfile.lock
  new file:   Procfile.dev
  new file:   app/controllers/inertia_example_controller.rb
  new file:   app/javascript/assets/inertia.svg
  new file:   app/javascript/assets/react.svg
  new file:   app/javascript/assets/vite_ruby.svg
  new file:   app/javascript/entrypoints/application.css
  new file:   app/javascript/entrypoints/application.js
  new file:   app/javascript/entrypoints/inertia.js
  new file:   app/javascript/pages/InertiaExample.jsx
  new file:   app/javascript/pages/InertiaExample.module.css
  modified:   app/views/layouts/application.html.erb
  modified:   bin/dev
  new file:   bin/vite
  modified:   config/initializers/content_security_policy.rb
  new file:   config/initializers/inertia_rails.rb
  modified:   config/routes.rb
  new file:   config/vite.json
  new file:   package-lock.json
  new file:   package.json
  new file:   vite.config.ts
```

There is an example controller that will display an Inertia welcome page at http://localhost:3100/inertia-example. But let's create our own controller and a page to see how that's done:

```ruby
# app/controllers/hello_controller.rb

class HomeController < ApplicationController
  def index
    render inertia: "Home", props: { greeting: "Hello from Rails 8 + Inertia + React" }
  end
end
```

The unique thing in the `index` action is the `render inertia:` line (we go into that next). This controller will render a React page (instead of the default `erb` view) and pass a `greeting` prop to that page. We create the React page under `app/javascript/pages`:

```Javascript
# app/javascript/pages/Home.jsx

import { Head } from "@inertiajs/react"

export default function Home({ greeting }) {
  return (
    <>
      <Head title="Home" />
      <h1>{greeting}</h1>
    </>
  )
}

```

One last things is to add  `root "home#index"` to `config/routes.rb`. Now, if we start the server with `bin/dev` and go to `localhost:3100`, we'll see the text "Hello from Rails 8 + Inertia + React". 

Tada 🎉 it works. But...how?

## Inertia Server Side: The `render inertia` Call

The Rails controller is using a new renderer called `inertia` and this works because the `inertia-rails` gem registers this renderer with Rail. We can see this in `lib/inertia_rails.rb` file in the Gem source code:

```ruby
ActionController::Renderers.add :inertia do |component, options|
  InertiaRails::Renderer.new(
    component,
    self,
    request,
    response,
    method(:render),
    **options
  ).render
end
```

The `ActionController.renderer.add` interface is provided by Rails, and it allows implementing custom renderers that can be used just like built-in renderers (i.e `render json`). Neat. 

The key to how Inertia does its magic is that the rendering is conditional based on an `X-Inertia` header. We can see that with `if @request.headers['X-Inertia']` in the Inertia gem source code [here](https://github.com/inertiajs/inertia-rails/blob/1d4a44c8af8f1668b4332c490e860fb21fcbfce7/lib/inertia_rails/renderer.rb#L40). If the request contains the header, then it's an Inertia XHR-like request and the controller will *render JSON*. If the request does not contain the header, then it will render HTML using `inertial.html.erb` template, which has a `div` with a `data-page` attribute that contains the React component name and props. Basically the JSON is stuffed into this `data-page` attribute when HTML is rendered. 

For example, when we load the `/inertia-example` page, the HTML response body looks like this:

```html
<body>
  <!-- BEGIN /Users/bhumi/.rbenv/versions/3.3.0/lib/ruby/gems/3.3.0/gems/inertia_rails-3.10.0/app/views/inertia.html.erb -->
  <div id="app"
       data-page='{
         "component": "InertiaExample",
         "props": {
           "name": "Bhumi"
         },
         "url": "/inertia-example",
         "version": "f33ca8e428f352821332f974d5d45830b30cc8f3",
         "encryptHistory": false,
         "clearHistory": false
       }'>
  </div>
  <!-- END /Users/bhumi/.rbenv/versions/3.3.0/lib/ruby/gems/3.3.0/gems/inertia_rails-3.10.0/app/views/inertia.html.erb -->
</body>

```

The interesting thing to note is that the *same* Rails controller will render JSON or HTML in a Rails app with the Inertia Rails adapter. 

## Inertia Client
On initial page load, Rails sends the above HTML, Inertia React clients boots, uses the React component and props from the embedded JSON and renders that React component *inside* the existing HTML.
    
The initial page looks fully server-rendered but is now a live React app. From this point on, clicks on Inertia links or form submissions are intercepted. Future navigations fetch JSON only and do not reload the whole page, using the `X-Inertia` header.

For an example of client side navigation with the header present, we can add an Inertia `Link` to the `Hello.jsx` and look at the HTTP request headers and response:

```ruby
import { Head } from "@inertiajs/react"
import { Link } from '@inertiajs/react';

export default function Home({ greeting }) {
  return (
    <>
      <Head title="Home" />
      <h1>{greeting}</h1>
      <Link href="/inertia-example">Go to Example</Link>
    </>
  )
}

```

When we click the link, we can see (in the "network" tab or dev tools) that the request headers and response headers both contain `x-inertia: true` and the server response is JSON:

```JSON
{
  "component": "InertiaExample",
  "props": {
    "name": "Bhumi"
  },
  "url": "/inertia-example",
  "version": "f33ca8e428f352821332f974d5d45830b30cc8f3",
  "encryptHistory": false,
  "clearHistory": false
}
```

So that's it. This is how Inertia does it's magic. We will stop here with this high-level overview in the interest of length (there is more to dig into with Inertia like forms, modals. Check out [Rails specific Inertia docs](https://inertia-rails.dev/guide/responses)for more Inertia features).

In summary, this is what happens during an Inertia client side navigation, when links are clicked or forms are submitted:

-  Inertia prevents the browser’s default behavior of doing a full page reload.
-  Instead, it sends an AJAX request (using `fetch` or `XMLHttpRequest`) to your Rails server, with the `X-Inertia` header.
- The Rails controller responds with JSON.
- The Inertia React client on the frontend parses the JSON and renders the React component (e.g. `InertiaExample`).
- Only the root `<div id="app">` content is updated, no full page reload.
- Browser history is automatically updated via the History API.

There are a number of moving pieces here. But the idea is to do all this while keeping Rails conventions and without the complexity of a full SPA front-end, if you don't need it. Of course, this approach has some limitations, Inertia may not be the right tool for multi-platform apps or apps that need a public API.  

Nevertheless, reasoning about how something like Inertia works behind the scene is interesting because it allows us to see what default Rails behavior can be customized (e.g.  `render`) while staying within the Rails conventions.

If you're using Inertia in your application, would be happy to hear about your experience - any tips or gotchas.