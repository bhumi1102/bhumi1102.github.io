---
layout: post
title:  "What is StimulusJS?"
---

This one is all about StimulusJS. We are continuing to survey the frontend landscape, coming back to the Rails ecosystem. I explored and wrote about htmx and alpine in previous posts (I also explored inertia.js though didn't make a post about it). 

Let's learn Stimulus with examples that illustrate its **main concepts: controllers, actions, targets, values, params and lifecycle callbacks**.

Stimulus is small enough that if you try the examples in this post, you will have good handle on how to use Stimulus controllers (even if you've never used it before).

## Context

Chances are you already know this, but here's some backstory so we're all starting on the same page. 

Stimulus is one of the libraries for the Hotwire (HTML-Over-The-Wire) approach (the other ones being Turbo and Strada). 

The key promise of Hotwire is to get the benefits of single page applications like faster, more fluid user interfaces *without* writing JavaScript. 

But we know that a little Javascript is still needed for modern web app behaviors like show/hide elements, etc. This is what Stimulus is for, for sprinkling in bits of JS as needed. 

Stimulus is advertised as a "modest Javascript framework for the HTML you already have". It has been around since 2017-2018. It's written in Typescript. The latest version, Stimulus 3.2, was released in August 2023. Stimulus has existed before the term Hotwire came on the scene.

Fun fact - Stimulus uses the browser's MutationObserver API to detect DOM changes (like when an attribute is modified or a child is added/removed from a node). It's cool, I won't include more here but if you desire to go down that rabbit hole, you can start with the MDN docs.

Installing and configuring Stimulus is pretty straightforward. In a Rails app, we can use `rails generate stimulus HelloWorld`. This creates a `hello_world_controller.js` file in `app/javascript/controllers` such that it'll be included in the usual builds, etc. Stimulus can be used outside of Rails as well. There is an npm package. And you can import into a `script` tag to play with it directly. 

With that preamble out of the way, let's get into the main concepts, starting with controllers, actions and targets.

## Controllers, Actions, Targets

Let's start with the canonical example: print a greeting when user clicks a button, along with the name that was typed into a text box. 

```html
<body>
  <div data-controller="hello">
    <input data-hello-target="name" type="text">
    <button data-action="click->hello#greet">Greet</button>
  </div>
</body>
```

This markup with `data-` attributes along with the following `hello_controller.js` is what's needed for Stimulus do its thing.

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  
  static targets = [ "name" ]
  
  greet() {
    const element = this.nameTarget
    const name = element.value
    console.log(`hello, ${name}!`)
  }
}
```

 `data-controller` attributes connects the HTML to the JavaScript object in hello_controller.js file.

 `data-action` attribute configures the `greet` method to be called when the button is clicked with the `click->hello#greet` .

Side note: a shortcut without the `click->` part, so just `data-action="hello#greet"`, works too. Because Stimulus defines default actions for some elements (i.e. `click` for a `button`). 

The `data-hello-target` attribute is a way to connect a given HTML element to the controller such that its value can be accessed inside the controller. 

In this case, with `data-hello-target="name"`, we add `name` to our controller’s list of target definitions. Stimulus *will automatically create* a `this.nameTarget` property which returns the first matching target element. We can use this property to read the element’s `value` and build our greeting string.

In general, what does the `static targets` line do? When Stimulus loads our controller class, it looks for a static array with the name `targets`. For each target name in the array, Stimulus adds 3 new properties to our controller. For the "name" target above, we get `this.nameTarget`, `this.nameTargets`, and `this.hasNameTarget`.

**Example: Copy to Clipboard Button**

You know the little copy button or icon next to text or code to make it easy to copy to clipboard. The below code builds that functionality in Stimulus using the browser's Clipboard API.

The HTML looks like this:
```html
<body>
  <div data-controller="clipboard">
    PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
    <button data-action="clipboard#copy">Copy to Clipboard</button>
  </div>
  More than one instance of the clipboard controller on the page
  <div data-controller="clipboard">
    PIN: <input data-clipboard-target="source" type="text" value="5678" readonly>
    <button data-action="clipboard#copy">Copy to Clipboard</button>
  </div>
</body>
```

The `clipboard_controller.js` looks like this:

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  
  static targets = [ "source" ]
  
  copy() {
     navigator.clipboard.writeText(this.sourceTarget.value)
  }
}
```

One interesting thing to note in the above example is that **we are using the same controller more than once on a page**. Stimulus controllers are reusable. Any time we want to provide a way to copy a bit of text to the clipboard, all we need is the markup on the page with the right `data-` annotations. And it just works.

In the HTML above, we have the exact same `div` for copying PINs duplicated twice. The 2nd copy has a different value so we can test that both copy button work and copy the right thing. The thing that's implicit here is that we have two different instances of the controller class, and each instance has its own `sourceTarget` property with the correct `value`. The controller is *scoped* to the `<div>`.

This implies that if we put two buttons inside the *same* `<div>`, things will *not* work as expect. The below will always copy the value in the *first* text box:

```html
<div data-controller="clipboard">
    PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
    <button data-action="clipboard#copy">Copy to Clipboard</button>
    PIN: <input data-clipboard-target="source" type="text" value="this won't get copied" readonly>
    <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

Alright, so far these examples demonstrates how *actions* and *targets* are used in the code. Let's see examples to the other Stimulus concepts - value and lifecycle callbacks.

## Value and Lifecycle Callbacks

Most JavaScript frameworks encourage you to keep *state* in JavaScript. Stimulus controllers are largely *stateless*. A Stimulus application’s state can live as *attributes in the DOM*. This is where the concept of `value` comes in. 

Stimulus controllers support typed `value` properties which automatically map to data attributes. `value` is a hash while `targets` are arrays. 

**Example: Slideshow**

Here is a slideshow controller that keeps the index of the currently selected slide in a `value` attribute. The HTML looks like this:

```html
<div data-controller="slideshow" data-slideshow-index-value="1">
    <button data-action="slideshow#previous"> ← </button>
    <button data-action="slideshow#next"> → </button>

    <div data-slideshow-target="slide">🐵</div>
    <div data-slideshow-target="slide">🙈</div>
    <div data-slideshow-target="slide">🙉</div>
    <div data-slideshow-target="slide">🙊</div>
  </div>
```

The `slideshow_controller.js` looks like this:

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "slide" ]

  static values = {index: Number}

  initialize() {
    this.showCurrentSlide()
  }

  next() {
    this.indexValue++
    this.showCurrentSlide()
  }

  previous() {
    this.indexValue--
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index != this.indexValue
    })
  }
}
```

When we add a value definition to our controller class like this `static values = { index: Number }`, Stimulus creates a `this.indexValue` controller property associated with a `data-slideshow-index-value` attribute (and handles the numeric conversion for us).

Note that Stimulus supports **lifecycle callback methods** for setting up or tearing down associated state when our controller enters or leaves the document, such as `initialize()`, `connect()`  and `disconnect()`

We can actually simplify the above controller with **Value change callback**. Notice that we manually call `this.showCurrentSlide()` method each time we change the value in `this.indexValue`. This is not needed. Stimulus will automatically do this for us if we add a method named `indexValueChanged()`. This method will be called at initialization and in response to any change to the `data-slideshow-index-value` attribute (including if we make changes to it in the dev tools). Once we add `indexValueChanged()` we can also remove the `initialize()` method altogether and the calls to `showCurrentSlide()` from `next()` and `previous()`. The simplified controller looks like this:

**params**

So far we have seen the concepts of *controllers*, *actions*, *targets*, *values* and lifecycle callbacks. There is one more feature: *params*. *params* are associated with the element and not 'attached' at the controller level, unlike *values* and *targets* (i.e. there is not a `static params = ` in the controller)

Here is an example:

```html
<div data-controller="content-loader">
    <a href="#" data-content-loader-url-param="/messages.html" data-action="content-loader#load">Messages</a>
    <a href="#" data-content-loader-url-param="/comments.html" data-action="content-loader#load">Comments</a>
</div>
```

That `-url-param` can be accessed in the controller's `load` action with `params.url`, like this:

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  load({ params }) {
    fetch(params.url)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
}
```

That's all for this one. We can wrap up now that we've covered the main concepts of Stimulus including controllers, actions, targets, values, params and lifecycle callbacks.


P.S. These examples are from the official Stimulus Handbook, which you can find [here](https://stimulus.hotwired.dev/handbook/introduction).