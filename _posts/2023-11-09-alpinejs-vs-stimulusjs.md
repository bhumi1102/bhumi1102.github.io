---
layout: post
title:  "What is alpine.js and how does it compare to stimulus.js?"
---

If you're a Rails developer and have heard of alpine.js but haven't had a chance to play with it, read on to get the lowdown in 5 mins! For this one, I build 4 UI things with alpine.js to get a feel for the main concepts.

I'm continuing to explore the frontend landscape for building modern interactive apps (in addition to Turbo and Stimulus, which are the Rails default for the HTML-over-the-wire approach) I was attracted to Alpine for words like 'lightweight' and 'minimal'. 

## What is alpine.js?

Alpine is advertised as "simple. lightweight. powerful as hell." Alpine adds behavior directly to the markup with a number of HTML attributes that start with `x-`. 

Before we get into the code I wonder who is behind Alpine? how long has it been around? 

An independent open source developer named Caleb Porzio is behind Alpine. He is also behind Livewire, which is the equivalent to Hotwire in the PHP/Laravel ecosystem. It seems Apline is commonly used alongside Livewire. From looking at github release history, Alpine has been around since 2019 and it has a few hundred contributors as well. 

Like I said, I was attracted to the word 'minimal'. Alpine is minimal in that it has 15 attributes, 6 properties, and 2 methods. I attempted to try out each of these (but will only highlight a select few to keep this post a good length).

Let's get into some code examples and see the `x-` attributes in action!

Note: btw you can install Alpine by including a `<script>` tag. No other configuration is needed.

**Code Examples**

One of the most common things to do in a web application UI is **toggling an element** based on some user input.  

```html
<div x-data="{ open: false }">
  <button x-on:click="open = ! open">Toggle</button>

  <div x-show="open" @click.outside="open = false">Content...</div>
</div>
```

In this example, we have a `button` that shows and hides a `div`. `x-data` is assigned a string that's evaluated as a JavaScript expression. In this case, we declare a JS object with the property `open`. Every property inside this object is made available to other `x-` directives inside the HTML element connected to `x-data`. 

`x-show` simply provides a way to show and hide DOM elements.

The `x-on` listens to dispatched DOM events and we can attach code to run for a given event by assigning a string of JavaScript with the event name like this `x-on:click=`.

Here's another example of a **simple counter** that uses `x-data`, `x-on`, and `x-text` to increment a number when a button is clicked.

```html
<div x-data="{ count: 0 }">
  <button @click="count++">Increment</button>

  <span x-text="count"></span>
</div>
```

`x-data` we've already seen. Same deal. It creates a JS object with one property `count`. 

The `@click=` is a shortcut for `x-on:click=`. Seems fine.

The `x-text` takes a JS expression and sets the text content of the HTML element with the result of the expression.  There is also `x-html` in Alpine that sets the "innerHTML" property of the element. 

The third example of **tabbed navigation** uses the `x-` directives we've already seen. Nothing new here.

```html
<div x-data="{ activeTab: 'tab1' }">
  <div class="tabs">
    <button @click="activeTab = 'tab1'" :class="{ 'active': activeTab === 'tab1' }">Tab 1</button>
    <button @click="activeTab = 'tab2'" :class="{ 'active': activeTab === 'tab2' }">Tab 2</button>
    <button @click="activeTab = 'tab3'" :class="{ 'active': activeTab === 'tab3' }">Tab 3</button>
  </div>
  <br>
  <div x-show="activeTab === 'tab1'">Some content for Tab 1</div>
  <div x-show="activeTab === 'tab2'">Hello from Tab 2!</div>
  <div x-show="activeTab === 'tab3'">We're inside Tab 3 Now!</div>
</div>
```

The last example is a **search bar** that uses two new directives `x-model` and `x-for`.

```html
<div
  x-data="{
      search: '',

      items: ['foo', 'bar', 'baz'],

      get filteredItems() {
          return this.items.filter(
              i => i.startsWith(this.search)
          )
      }
  }"
>
<input x-model="search" placeholder="Search...">

<ul>
    <template x-for="item in filteredItems" :key="item">
        <li x-text="item"></li>
    </template>
</ul>
</div>
```

Here we see a for loop with `x-for`. Nothing fancy there. 

The JavaScript with `x-data` in this example is a bit more elaborate. We create an object with a couple of properties and a function that's called to do the actual searching. I can imagine the formatting getting messy if there is any more JS than this, an no syntax highlighting for strings of code. 

The new thing is `x-model`. `x-model` is meant to *bind* the value of an input element to a piece of (internal) data in Alpine. This is how you'd get the (very demonstrable 'reactive') behavior of typing in a text field and seeing whatever you type displayed in an element on the page. Nice.

## How does Alpine compare to Stimulus?

My main motivating questions for this post were along the lines of: how does alpine.js compare to stimulus.js? Are they interchangeable? Is one suited for X and the other one better at Y? What's it like to use alphine.js with a Rails application?  

And here's what I learned in terms of answers:

Stimulus uses data attributes and controllers to define and manage interactivity. You write your JavaScript inside Stimulus controllers. Alpine uses HTML attributes to add behavior to DOM elements. YOu don't write much JS (you don't have a separate `.js` file). Though when you do write bits of JS, it's code in strings assigned to the various directives like `x-data`. You could probably use Alpine and Stimulus interchangeably. Alpine is probably better suited for smaller size apps. Stimulus does focus on organizing its logic in components. Using Alpine with a Rails project is pretty simple, it doesn't require any special configuration. 

So now you know the core concepts of Alpine and how it compares to Stimulus . If you've worked with Alpine, do share your experience. I would love to learn from you too.


P.S. I couldn't get the examples using `$refs` in Alpine docs to work (tried multiple browsers). I'm not ruling out user error. But the thing that stuck out to me is that there are no error messages or anything. While Alpine is simple, lack of hints for debugging could be a bummer when things don't 'just work' as expected.