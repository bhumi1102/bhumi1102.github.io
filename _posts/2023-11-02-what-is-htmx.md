---
layout: post
title:  "What is HTMX?"
---

The motivation behind this post is to explore the frontend technology landscape for building modern interactive apps with Ruby and Rails. I will dive deep into Hotwire and Turbo but first I wanted play with `htmx`. 

## HTMX: How does it work? Where did it come from?

HTMX stands for HyperText Markup eXtensions. The pitch is that "it allows you to create interactive web applications with minimal javascript". And "it enables you to update parts of the page dynamically, without the need for full-page reloads". Sounds familiar right?

HTMX uses HTML attributes and HTTP requests to achieve this. Specifically htmx gives us access to AJAX and CSS transitions using HTML attributes (It has experimental support for web sockets and server sent events as well).

The motivating questions for why htmx exist: What if elements other than `<a>` and `<form>` could make HTTP requests? What if any event could trigger HTTP request, not just 'click' or 'submit'? And what if the HTTP response did not replace the *entire* screen always? In other words, what if we remove these 'arbitrary constraints' from HTML?

This statement from htmx site "by removing these arbitrary constraints, htmx completes HTML as a hypertext." leads me to think that it's possible that future versions of HTML *could* directly incorporate these features from htmx. It's possible. Though a big enough change that maybe in a decade we'll see if it goes this way.

Anyway, htmx is created / maintained by an organization called bigskysoftware on github and they are in Montana in US apparently. The creators behind htmx have written a book. HTMX is the successor for intercooler.js (if that means something to you, I had not heard of it). Intercooler has been around since 2013.

Note: I had not worked with HTMX until recently. It's something I've been curious about so I spent a few hours exploring what HTMX can do. I'm sharing what I learned. If you're more familiar with HTMX, do share your experience. Especially how it relates to other frontend options and patterns.

That'll do for the preamble. Let's dive in and see some code in action.

## HTMX Code Examples

The most basic demonstration of htmx is using HTML attributes like `hx-get` or `hx-post` to make an **AJAX request to the server**. Then the server HTTP response updates the `hx-target` element.

```html
<button hx-get="/load_data" hx-target="#output">Load Data
</button>
<div id="output"></div>
```

In this case, I simply sent back some plaintext from the server. Clicking the button renders that text within the `div`. 

```ruby
def load_data
  render plain: "Loaded data via HTMX at #{Time.now}"
end
```

Another htmx example involves **CSS transitions**.  Here is an example of fading in an element when the user clicks a button. 

```html
<button id="fade-me-in"
        hx-get="/fade_in_demo"
        hx-swap="outerHTML settle:1s">
        Fade Me In
</button>

<style>
  #fade-me-in.htmx-added {
    opacity: 0;
  }
  #fade-me-in {
    opacity: 1;
    transition: opacity 1s ease-out;
  }
</style>
```

The server sends an element that we want to fade in. In this case, it's important that the `id` is the same between the original and the new content. 

```html
<div id="fade-me-in" class="htmx-added">
  This content fades in using HTMX!
</div>
```

Another example is inline editing a form. The form below has a button that makes a request to the server at `contact/1/edit`. The controller needs to respond with the edit form.

```html
<div hx-target="this" hx-swap="outerHTML">
    <div><label>First Name</label>: Joe</div>
    <div><label>Last Name</label>: Blow</div>
    <div><label>Email</label>: joe@blow.com</div>
    <button hx-get="/contact/1/edit" class="btn btn-primary">
    Click To Edit
    </button>
</div>
```

That give us an idea of what working with htmx looks like. There are many more examples on the official htmx site.

Zooming out a little bit. I'll share answers to some questions I had written down:

**Q: So where does htmx fit in in the frontend technology landscape?** 

A: It's definitely meant to support server side rendering. Encourages passing HTML over the wire (as in Hotwire) and keeping business logic on the server. People also mention (on hackernews) that HTMX and Web Components can work well together. Though I'm not familiar with Web Components (so it'll have to be a topic for a future post).

**Q: How does it relate to stimulus.js? Does it work with it?**

A:  There is another library called `hyperscript` that is htmx's analogue to stimulus.js. It's not part of htmx but an "official companion project" (though it's under construction). 

**Q: Do people use htmx with Rails? with Hotwire / Turbo?**

A: So it looks like htmx is sort of a competitor to Hotwire / Turbo. I deduce this from the fact that there is a document titled "Hotwire / Turbo to htmx Migration Guide" on the official htmx site. The details are a little sparse but here is an example of a common feature. `hx-boost` sounds equivalent to Turbo Drive.  

>"The `hx-boost` attribute allows you to “boost” normal anchors and form tags to use AJAX instead." For anchor tags, ...will push the url [specified in the href] so that a history entry is created...

It seems to me that htmx is a bit of 'unbundling' of Hotwire? It's a less complete frontend solution than Hotwire but some may prefer it as it's less opinionated and more of a mix-and-match.

There is a bit of cognitive load with using Turbo Frames and Turbo Streams. That makes sense as they give you more fine-grain control over partially updating different elements on the page and synchronizing with server state. The upcoming 'morph' feature with Turbo 8 seems to promise to take away some of that cognitive load from the developer. You automatically get the partial page updates in the cases where it makes sense and broadcasting server state happens by default (so you, as the developer, don't need to think it, unless you need to higher UI fidelity than what the 'happy path' offers). That's what I understand at least, though I'll need to play with Turbo 8 morph and report back.

To wrap up, after playing with htmx this week, my mental model is this: HTMX focuses on synchronizing state with the server and UI updates with CSS using HTML attributes and no (custom) Javascript. 

Have you used htmx? If so, what is a use case where it fits in nicely? Feel free reach out and share your experience with HTMX if it's something you've explored.


P.S I shared a little clip with HTMX examples [here](https://twitter.com/bhumi1102/status/1719419572560732663) 