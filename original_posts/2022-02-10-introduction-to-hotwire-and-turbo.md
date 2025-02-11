layout: post
title:  "Introduction to Hotwire and Turbo"
---

The Rails 7 release in December 2021 officially shipped with Hotwire. Or more precisely with the turbo-rails and turbo-stimulus gems. Hotwire is an 'umbrella' term for the concept of sending Html-Over-The-wire but not a specific library.

Although I started working with Ruby and Rails in 2012 when I transitioned doing web development, I have not done Rails in production since Rails 4/Rails 5. I came back to Rails for side projects last year and have been following along with the New Magic announcements. I decided to dive-in and learn more about the motivation and concepts of Hotwire. This writing is to summarize what I learned. 

The following writing is for wrapping our head around the *concepts*. So you can sit back with your coffee/tea and just read, not much code to type-along.

Note: You do not need to be Ruby on Rails developer to read this. The ideas around Hotwire and server-side rendering are making waves across different web development stacks - PHP/Laravel, Elixir/Pheonix, Python/Dyango have equivalent tech that implements Hotwire.

## <a name="intro"></a> What is Hotwire and What is Turbo

HTML-over-the-wire or Hotwire is a *technique* for building web applications. It's not one technology, it's an umbrella term for Turbo, StimulusJS, and Strada (in the Rails world). Turbo *is* the actual technology that makes the Hotwire approach possible. Turbo is the actual [code](https://github.com/hotwired/turbo). 

Yes Turbo is created by the team behind Ruby on Rails and it's an evolution of Turbolinks with bunch of other functionality added.  But Turbo can be adapted to other languages and backend frameworks (and [already](https://github.com/tonysm/turbo-laravel) [has](https://github.com/hotwire-django/turbo-django)!).

**So what does the Turbo code do? What problem does it solve? How does it help us?**

A little history - Turbo descends from pjax (from github). pjax was a JQuery plugin that worked by fetching HTML from the server via ajax and replacing the content of an element on a page with the loaded HTML without a full page reload. The key idea being the 'without full page reload' part so that it feels fast to the user.

The reason full-page refreshes often feel slow is *not* because the browser has to process a bunch of HTML sent from a server. Browsers are fast at that. It's because CSS and JavaScript has to be reinitialized and reapplied to the page again. Even if the files are cached. **So Turbo's job then is to get around this reinitialization. It does this by maintaining a persistent process (similar to SPA, but an 'invisible' one)**. It intercepts links and loads new pages via Ajax. Server returns fully formated HTML documents.

Officially "Turbo is a collection of techniques for creating fast, progressively enhanced web applications without using much JavaScript. All the logic lives on the server, and the browser deals just with the final HTML."

This approach can, of course, be contrasted with the Single Page Application (SPA) approach. Where we'd get JSON from the server, use client-side JavaScript - like a boatload of JS involving frameworks and libraries that need to keep state and do routing - to eventually transform that JSON data into DOM updates.

**The promise of Turbo is writing less JavaScript** and more of your preferred backend language - Ruby or PHP or Python or what have you. All of your logic for interacting with your domain model and other business logic can live in one place, on your server. **The other promise is to sidestep the complexity** of full-fledge client-side JavaScript frameworks and the associated tooling (*cough* webpack). The only JavaScript you need is Turbo.js

Of course we'd want to **get these benefits without sacrificing any of the speed or responsiveness** associated with modern web applications with rich user experiences. And that is just what hotwire.dev promises.

If we go all the back to AJAX, the official documentation (and by that I mean wikipedia) says 

"With Ajax, web applications can *send and retrieve data from a server asynchronously* (in the background) without interfering with the display and behavior of the existing page...Ajax allows web pages and, by extension, web applications, to *change content dynamically without the need to reload the entire page*." 

So in other words, the big promise of AJAX was to 'update content *without* full-page reload'. So that it feels smooth and fast for the user. That is *still* what we are after. With Turbo we can do this in a simpler, more elegant way and automatically get the benefits by including Turbo.js and following some conventions (without writing any custom JavaScript!).

Turbo does its job with the following components: 

**Turbo Drive** speeds up links and form submissions by not requiring page reloads.

**Turbo Frames** decompose pages into independent contexts, which scope navigation and can be lazily loaded.

**Turbo Streams** deliver page changes over WebSocket, SSE or in response to form submissions using just HTML and a set of CRUD-like actions.

[aside] SSE is server-sent events. The major difference between WebSockets and Server-Sent Events is that WebSockets are bidirectional (allowing communication between the client and the server) while SSEs are one-directional (only allowing the client to receive data from the server).

**Turbo Native** lets your majestic monolith form the center of your native iOS and Android apps, with seamless transitions between web and native sections.

Turbo 7.1 was released November 24, 2021 (keeping the Turbolinks versions) and Hotwire ships by default in Rails 7, released December 15, 2021. 

Let's see how each of the 4 Turbo components work. We also cover some constraints and nuances to keep in mind when building applications with Turbo.

## Turbo Drive: Navigate Within a Persistent Process

Turbo Drive gives us the same speed of SPA by using the same persistent process. The persistent process is managed by Turbo (there is no client-side router, no state to carefully manage). 

**Following Links**

This works by intercepting all clicks on `<a href>` links to the same domain. When you click an eligible link, Turbo Drive prevents the browser from following it, changes the browser’s URL using the History API, requests the new page using fetch, and then renders the HTML response.

[aside] History API - allows manipulation of the browser *session history*, that is the pages visited in the tab or frame that the current page is loaded in. I see the `pushState()` function is part of this API (which is what `pjax` used to update browser URL, etc.)

**Form Submission**

During rendering, Turbo Drive replaces the current `<body>` element outright and merges the contents of the `<head>` element. The JavaScript `window` and `document` objects, and the `<html>` element, persist from one rendering to the next.

The speed with Turbo happens free just by following a few conventions. Though it is possible to interact directly with Turbo Drive to control how visits happen or hook into the lifecycle of the request.

## Turbo Frames: Decompose Complex Pages

Turbo Frames is a way to scope independent segments of a page inside a `turbo-frame` element such that they can be lazily loaded and their navigation be scoped. Scoped navigation means all interaction within a frame (e.g. clicking links, submitting forms) happen within that frame, keeping rest of the page from reloading.

I heard DHH say on Remote Ruby interview that the origins of Turbo Frames are based on making mobile stuff work for Hey email.

[aside] the use case for which this is designed, reminds me of the 'aync dashboard' work I did to speed up the Dashboard of our web app at Castlight. Loading bits of the page 'later', in parallel instead of in-band with the initial page load. I recall using .js.erb files I think and looking at the network tab a lot.

**Scoped Navigation**

```html
<turbo-frame id="new_message">
  <form action="/messages" method="post">
    ...
  </form>
</turbo-frame>
```

When we submit the form above, Turbo extracts the matching `turbo-frame` element with the `id` of `new_messages` from the HTML response and swaps its content into the existing `new_message` frame element. The rest of the page stays as it was.

**Deferred Loading**

```html
<turbo-frame id="messages" src="/messages">
  <p>This message will be replaced by the response from /messages.</p>
</turbo-frame>
```

To differ loading we add a `src` attribute to the `turbo-frame` element. The HTML response from the URL value will be used to automatically load content into the matching frame `id`. 

**Are Turbo Frames like iframes then?**

They sound like iframes but no Turbo frames are part of the same DOM, styled by the same CSS, and part of the same JavaScript context. So they don't suffer the weirdness associated with iframes.

Other advantages of Turbo Frames:
1. Efficient caching - each segment is cached independently, so you get longer-lived caches with fewer dependent keys.
2. Parallelized execution - each defer-loaded frame is generated by its own HTTP request/response and handled by a separate process. So different segments on a page load in parallel without having to manage the process.
3. Ready for mobile - Each segment can appear in native sheets and screens without alteration, since they all have independent URLs.

## Turbo Streams: Deliver Live Page Changes

While Turbo Frames give us partial page updates in response to direct interactions within a single frame (link clicks, form submits), Turbo Streams let us change any part of the page in response to updates from a WebSocket connection (or SSE).

[aside] Streams is a conceptual continuation of what was first called RJS and then SJR.
- The big idea around [RJS](https://rubyonrails.org/2006/3/28/rails-1-1-rjs-active-record-respond_to-integration-tests-and-500-other-things), JavaScript written in Ruby, from that 2006 post announcing Rails 1.1 is that you don't have to write JavaScript to Ajaxify things in Rails, you can write Ruby. (So yeah, we are still after the same thing!).
- The idea behind Server-generated Javascript Response [SJR](https://signalvnoise.com/posts/3697-server-generated-javascript-responses) from this post from 2013 is pretty reminiscent of what we have with Turbo. The flow is something like this - a form is submitted via AJAX request, server generates a JavaScript response that includes an updated HTML template, client evaluates the JavaScript returned by the server, which then updates the DOM. We can see how Streams is a conceptual continuation of this flow with eliminating more JavaScript. RJS was a poor man's CoffeeScript and it turned people away from the concept of server-generated JavaScript, but in 2013 rails re-committed to SJR while leaving behind RJS. The last sentence of that 2013 post says:

*"The combination of Russian Doll-caching, Turbolinks, and SJR is an incredibly powerful cocktail for making fast, modern, and beautifully coded web applications. Enjoy!"*

Unlike RJS and SJR it's not possible to send custom JavaScript as part of Turbo Stream actions, by design! Turbo focuses on sending HTML and updating the DOM and then if needed we can connect additional behavior using Stimulus actions and lifecycle callbacks.

### How It Works?

Turbo Streams introduces a `<turbo-stream>` element with an `action` and a `target` attribute. The actions could be append, prepend, replace, update, remove, before, after. We include the HTML to insert or replace in a `template` tag and Turbo does the rest.

[aside] [HTML template tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template) is a way for holding HTML that is not to be rendered immediately when a page is loaded but may be added to the page later using JavaScript.

### Advantages of Turbo Streams

1. Reuse server-side templates - same templates that created the first-load page are used to generate live page updates (in practice some CSS is needed to show/hide certain elements in the two cases).
2. HTML over the wire - don't need any client-side JavaScript (other than Turbo.js) and save all the effort it takes to fetch JSON and turn it into HTML.
3. Simpler control flow - clear to follow what happens in response to WebSocket or SSE by looking at the HTML to be changed. No indirection with client-side routing, event bubbling and such.

## Turbo Native: Hybrid apps for iOS and Android

I am not a mobile developer. But here are some points I took away from reading about Turbo Native.

- Ideal for building hybrid apps where server-rendered HTML can be used to get baseline coverage of your app's functionality in a native wrapper. Along with a few native screens that can really benefit from the high fidelity.
- Going hybrid allows you the freedom to upgrade your app without going through the slow app store release process. Anything done in HTML can be changed in your web application and be instantly available to users.
- Turbo Native is not a framework that abstracts native APIs. It also does not try to create native code that's sharable across platforms. (In other words, you still need native devs for iOS and Android, but those devs just have less work do). The part that's sharable is the HTML that's rendered server-side.

---

So that covers the main concepts around Turbo Drive, Turbo Frames and Turbo Streams. Let's learn about some nuances to keep in mind when building applications with Turbo.

Also by the way, while Turbo is not Rails specific, [turbo-rails gem](https://github.com/hotwired/turbo-rails) is the reference implementation of Turbo for Ruby on Rails. The Turbo documentation says you don't need a backend framework to use Turbo. (I don't quite get that, doens't the backend need to return HTML with the appropriate elements `<turbo-stream>`, `<turbo-frame>` along with the correct `ids` to replace the right things on the page.) 

---

## Constraints and Nuances of Building Turbo Applications

Since there is no full page load, the JavaScript `window` and `document` objects retain their sate across page changes (any other objects we leave in memory will stay in memory). We can't rely of full-page reload to reset our environment, our application is a persistent, long-running process in the browser. Below are things we need to be aware of to design our application to gracefully handle this constraint:

### 1. Script Elements

When navigating with Turbo, `<script>` elements in the `<head>` are okay, Turbo Drive appends them to the current `<head>` and the browser loads and evaluates them. 

`<script>` elements in the `<body>` are not good. To install behavior, or to perform more complex operations when the page changes, avoid script elements and use the `turbo:load` event instead.

For `<script>` elements we don't want Turbo to evaluate after rendering, annotate them with `data-turbo-eval="false"`. The browser will still evaluate those scripts on the initial page load.

For loading our application's JavaScript bundle, do this in the `<head>` tag always. Also if we add a fingerprint to each script and `data-turbo-track="reload"` attribute, Turbo can force a full page reload when we deploy a new JavaScript bundle.

### 2. Caching

Let's define some terms like Restoration visits and Application visits first.

- Restoration visits are when you navigate with the Browser's back or forward button. Turbo Drive renders a copy of the page from the cache if possible. (Drive also saves the scroll position of each page before navigating away and automatically returns to this saved position. nice)
- Application visits are the onces initiated by clicking a Turbo Drive-enabled link or programmatically calling `Turbo.visit(location)`. Always issues a network request. These result in a change to browser's history, Turbo Drives pushes a new entry onto the browser's history stack using `history.pushState`. `data-turbo-action` is the attribute to decide what type of visit it is, default is `advance`, it could also be `replace`.
- When Turbo Drive saves a copy of the current page to cache, it uses `cloneNode(true)`, which apparently means attached event listeners and data are discarded. (`cloneNode` clones the DOM node)

**Preparing the Page to be cached** - we can listen to the `turbo:before-cache` event with `document.addEventListener` if we need to do things to the document before Turbo caches it. For example, reset forms, collapse UI elements, etc. so the page is ready to be displayed again.

**Detecting when a preview is visible** - Turbo Drive adds a `data-turbo-preview` attribute to the `<html>` element when it displays a page from cache. We can check for the presence of this if we want to enable/disable behavior.

**Opting out of caching** - We can control caching behavior on a per-page basis by including a `meta` element in the page's `head`. Like this:

```html
<head>
  ...
  <meta name="turbo-cache-control" content="no-cache">
</head>
```

### 3. Installing JavaScript Behavior

Since the usual `window.onload`, `DOMContentLoadeded`, or JQuery `ready` events will only fire after initial page load, we need a strategy for installing JavaScript behavior on Turbo page loads. There are two options:

#### Observing Navigation Events

There is an event, `turbo:load` that fires after the initial page load and again after every Turbo Drive visit. We can use this like this:

```html
document.addEventListener("turbo:load", function() {
  // ...
})
```

- avoid using `turbo:load` event to add other event listeners directly to elements on the page body. Instead use [event delegation](https://learn.jquery.com/events/event-delegation/) to register event listeners once on `document` or `window`.

#### Attaching Behavior with Stimulus

New DOM elements can appear on the page at any time from frame navigation, stream messages, client-side rendering, or Turbo Drive page loads. Stimulus with its lifecycle callbacks and conventions can handle all of these in a single place.

**How Stimulus Works** - it connects and disconnects its controller and event handlers whenever the document changes using the [MutationObserver API](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver). This allows it to handle all types of DOM updates.

### 4. Making Transformations Idempotent

The context here is performing client-side transformations to HTML received from the server. The example we can imagine is grouping a collection of elements by date using browser's knowledge of current time zone. 

[aside] Making something, a function, idempotent means that regardless of how many times we apply the function to a given input the result will be the same as applying it just once. Meaning there are no more changes beyond its initial application.

We have to think about this because of caching. "Consider what happens if you’ve configured this function to run on turbo:load. When you navigate to the page, your function inserts date headers. Navigate away, and Turbo Drive saves a copy of the transformed page to its cache. Now press the Back button—Turbo Drive restores the page, fires turbo:load again, and your function inserts a second set of date headers."

We can solve this by detecting if the transformation is already present on the HTML before adding it. In this we'd check for the presence of a date divider. (We can also add a flag via `data` attribute but that's less robust)

### 5. Persisting Elements Across Page Loads

The context here is that we may not want certain elements to change across page loads. Turbo Drive allows us to mark them *permanent*. The example we can imagine is a shopping cart icon with a counter. We don't want the counter to change to a previous/lower number if the user navigates with a back button after adding an item to the cart on the current page.

We do this by marking the counter element permanent. 

```html
<div id="cart-counter" data-turbo-permanent>1 item</div>
```

The HTML id is needed as Turbo Drive matches all permanent element by ID before rendering, and transfers them from the original page to the new page. Preserving data and event listeners.

---
## Summary

That's all, that covers most of the concepts for understanding Turbo and Hotwire. It's all about updating things on the page in a way that feels fast and smooth to the user, without needing to write a lot of client-side Javascript.

**HTML-over-the-wire or Hotwire** is a concept, not a specific technology. The main idea is to send server-rendered HTML to the client, instead of JSON, leaving application logic to the server. 

**Turbo** is the technology that implements Hotwire in the Rails world. And other backend stacks have their own version of Turbo. Turbo consists of Drive, Frames, and Streams. 

**Turbo Drive** is an evolution of Turbolinks. When enabled, it intercepts link clicks and form submission, maintains a persistent process that leaves `window` and `document` unchanged, but replaces the `<body>` element of the response. 

**Turbo Frames** allow different sections of the page to be lazily loaded and elements on the page to be automatically scoped by using `<turbo-frame>` element so that a response updates only the frame with a matching `id`, leaving rest of the page untouched. Turbo Frames are similar to `iframes` but different, in that they're part of the same DOM, styled by the same CSS, and part of the same JavaScript context.

**Turbo Streams** take things even further. They allow for partial page updates from not only user interactions (like form submission) but also in response to websocket and SSE. `<turbo-stream>` elements support automatically changing the HTML of a target element by various operations like replace, append, prepend, remove, etc.

**When building applications with Turbo** we have to keep in mind that our application is a persistent, long-running process in the browser. We can't rely of full-page reload to reset our environment and have to approach a few things with care, like caching, script element, and installing JavaScript behavior, etc.

Finally, it's worth noting that Hotwire and Turbo approach is not suitable for *all* web applications. Sometimes the complexity of client-side rendering with a JavaScript SPA is needed for a high-fidelity user experience of text editor, for example. (such as the Trix editor from the Rails community). And yet, for many modern web applications it would be worthwhile to reach for Hotwire/Turbo for the conceptual compression of complexity offered.