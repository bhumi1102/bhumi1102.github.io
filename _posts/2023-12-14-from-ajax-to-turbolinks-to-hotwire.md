---
layout: post
title:  "How did we go from AJAX to Turbolinks to Hotwire? A brief web history"
---

This post is a bit different. It has more words, no code blocks, and is a bit longer. I'm aiming to cover last 10+ years of web history as succinctly as possible. Let's go. 

In order to appreciate a new technology solution, we have to remind ourselves of the problem sometimes. In web application development, we've always wanted to update the page in the browser, in a way that feels fast and seamless for the end user, based on 1. user interactions - clicking things, submitting forms 2. some data changing on the server. 

Our starting point was a full HTTP request/response cycle, with a round trip to the server and a full rendering of the response by the browser. When that paradigm wasn't fast and fluid enough for the user, our options for a solution were "not doing a full page reload" and "fetching the data from the server in the background".

AJAX was an exciting game changing technology along these lines in the early 2000s. So I hear. I wasn't doing web dev work back then. Though I wish I was, it would've been cool to experience that first-hand.

An aside: This is history lesson is relevant to me because I am trying to fill in my "rails: the missing years". Someone tweeted how they're glad they started using ruby/rails a long time ago so now it's easier to keep up with the incremental changes. While I started with Rails in 2012 I didn't get to keep using it continuously. My company moved to SOA with Java/Spring services with Angular and bunch of other things. We still maintained a Rails monolith but it was 'frozen' in time. I do remember using RailsUJS and writing many `.js.erb` files. I missed the Turbolinks era. I come back to Rails in 2020/21 around the time Hotwire was referred to as "new magic".

Anyway, that's my Rails history. But let's get back to web history, starting with AJAX and RailsUJS!

## AJAX and RailsUJS

The big idea with AJAX was that web applications can send and retrieve data from a server in the background, asynchronously, without interfering with the existing page the user is looking at. And then change the content on the page dynamically, without the need to reload the entire page. 

In practice, there are two ways to make AJAX requests from the client JavaScript. `XMLHTTPRequest` and `fetch`. XMLHttpRequest is the old way using callbacks. Fetch API is the newer more powerful way and it uses promises. Here's an example `GET` request with `fetch`:

```javascript
async function logMovies() {
  const response = await fetch("http://example.com/movies.json");
  const movies = await response.json();
  console.log(movies);
}
```

So we can make an AJAX request to the server in the background. We do need some JS on the client to handle the response though, in order to prevent a full page reload. (i.e. if the server sends back a full HTML page and the client doesn't do anything fancy, there will a be full page reload from the AJAX request). 

Within Rails, the answer was: what if we send little bit of JavaScript in the AJAX response from the server that tells the browser what to update on the page? When the browser executes that JavaScript the net result will be updating just the parts of the page that we want. **That was the idea behind RailsUJS** (which is a [single js file](https://github.com/rails/rails/blob/main/actionview/app/assets/javascripts/rails-ujs.js) btw that was moved into Rails with Rails 5).

We used `data-remote = true` to send AJAX requests in Rails for link clinks and form submissions. And I vividly remember writing those `.js.erb` templates. And feeling powerful, like I'm orchestrating this dance between the client and the server and it worked. But at what cost? 

Well, we have to write JavaScript and put it in our view templates and still have HTML templates for non-AJAX requests. In the controller we have to deal with returning multiple formats in `respond_to`.

At this point in the timeline, there was a fork in the road for web applications. One path lead to doubling down on JavaScript and the other path lead to going *back* to returning HTML from the server.

## A fork in the road

Path 1: JavaScript. The thinking was JS, more and more JS in the server response. This path started with a mix of client-side rendering and server-side rendering. Some requests returned HTML and other requests returned JS or JSON. 

Eventually leading to "what if we send *all* of the JS for the application in one request upfront?" AKA the Single Page Application (SPA) approach. Which is all client-side rendering (plus routing and state management) while the server acts as an API and returns JSON data. For Rails, this meant the introduction of Rails api-only mode. Embracing all the JavaScript with Webpacker default, etc.

The downside of the SPA approach is well appreciated by anyone reading this so I'll breeze through that now and move on.

Path2: HTML. The other line of thinking was "hold on. Instead of more and more JS, what if we can go back to returning HTML". We just need to find a way to *not* do a full page reload as a result to receiving an HTML response from the server to an AJAX request.

Cue Turbolinks.

## Turbolinks and Hotwire

Browsers are fast at rendering HTML. Reprocessing the CSS and JS on the page is what makes it feel slow. So Turbolinks will *replace* the `<body>` and *merge* the `<head>` from the HTML response. No reloading CSS and JS if `<head>` hasn't changed.

Turbolinks also updated the browser history so the back/forward buttons work. And cached pages so the back button can display the cached page while the new one is being loaded, etc.

However, the response to Turbolinks was lukewarm (that's my sense at least, it was during my "missing years"). Likely because **Turbolinks was a good idea that didn't go *far enough***. Until Hotwire!

With Turbolinks, we were replacing the the entire `<body>`, so the natural question is "what about just updating specific parts of a page?"

And that's the main idea behind Turbo. With Turbo Frames and Streams we have a way to mark specific elements of a page that need to change based on user interaction (plus broadcast updates from the server when data changes). 

And Turbo Drive takes Turbolinks farther by intercepting form submissions in addition to link clicks.

The tagline for Hotwire is "speed and responsiveness of an SPA, without the complexity of an SPA, and without writing custom JS." No brainer right.

There is more to say about Turbo and Hotwire of course but we'll get into that in future posts.

## Where are we going

Another motivation for zooming out and looking at history is that knowing how we got *here* can allow us to predict where we're *going*. What is the natural progression in web development from here? 

Right now, we have Turbo Drive, Turbo Frames, Turbo Streams, Turbo Page Refresh/Morph on the web (also Turbo Native and Strada and what not). Can't blame someone for wondering "how many Hotwire things do I gotta learn?!" 

I can imagine a future where this categorization goes away. Instead we have 2 things. 

We have 'manual' tools like Turbo Streams that give us a more fine-grain control for when we need to do some precise surgery to the page. More powerful but require more cognitive load to use. 

And we have 'automatic' tools. The stuff we get 'for free'. Turbo Drive, which doesn't require code changes to use. And the page refresh and morphing coming out in Turbo 8 is one example. It doesn't require annotating the code in special ways. It just works.

I see those as the two main "things". I dunno. Hotwire is a big leap in brining back simplicity and developer happiness in web development. I can see a future where it's more prevalent, even outside of the Rails ecosystem. How Hotwire is currently presented probably needs some iterations though, in order for that to happen (Branding and packaging/positioning is hard and naming things is hard too).

While we wait to see how the future unfolds, in the meantime, I'd like to explore what Hotwire enables right now. I'll start with "here's what Turbo can do." But then quickly turn to "here's the UI I want to build, can I use Turbo for that and how?" Also what are the limitations? where using Turbo and Stimulus still feels wonky? 

For now, we can wrap up this web history lesson of how we got from AJAX/RailsUJS to Turbolinks to Hotwire. There are many details I didn't cover as I going for a 'brief history'. This perspective is based on a mixture of first-hand experience and what I've surmised from the sidelines. 

If you've been a Rails developer through all of these transitions and want to add something from your experience, I'd be curious to hear from you.


P.S. There are many parallel technologies that exist along this timeline of RailsUJS to Turbolinks to Hotwire. That contributed to the progression from complex to simple. In the Rails ecosystem, StimulusReflex and CableReady have been around for several years. In other ecosystems, there is Phoenix LiveView and Laravel Livewire. And even SPA frameworks have contributed to the evolution in web development (sometimes you have to experience the pain before you reach for the remedy).  