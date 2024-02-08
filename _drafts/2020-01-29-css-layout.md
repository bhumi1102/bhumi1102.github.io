---
layout: post
title:  "CSS learnings and reference - flexbox, grid, gradient, header, footer"
date:   2020-02-03
---

**Flexbox** is a one-dimensional layout method for laying out items in rows or columns. Items flex to fill additional space and shrink to fit into smaller spaces.

This is an evolution from the way layout was done in limited and frustrating way - by float and positioning (static, relative, absolute, etc.)

**CSS Grid** on the other hand is more for two-dimentional layout. "CSS Grid Layout excels at dividing a page into major regions or defining the relationship in terms of size, position, and layer". 

This is an evolution of for layouts people did on top of HTML tables.

So Grid system seems more powerful than Flexbox and kind of a superset of Flexbox

This is what I get when I use **Bootstrap's default Grid System:**
>Use our powerful mobile-first flexbox grid to build layouts of all shapes and sizes thanks to a twelve column system, five default responsive tiers, Sass variables and mixins, and dozens of predefined classes

So this says it's based on Flexbox (but uses the word Grid so a little confusing). 
Read this explanation of [Flexbox from css-tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/#flexbox-background)

**SCSS adding padding to h1 on home**
Making a change to home.scss file didn't seem to do anything. So..

Side note - some history of scss and ruby-sass and how it was a thing when ruby was popular but now since Node.js is more popular (and faster, gasp!) they moved on to DartSass and LibSass. More [here](https://sass-lang.com/ruby-sass)

I added this from some SO post and that didn't do anything:
`/*
 *= require_tree .
 *= require_self
 */`

Solved: by explicitely importing the individual style sheets like this:
`@import "home";
@import "dashboard";`


**Background Gradient image**
I used this color picker to [generate the CSS](https://mycolor.space/)
Adding this is needed to 'stretch' the image to fit the entire screen, otherwise it only fits the height of the content (from this [SO](https://stackoverflow.com/questions/2869212/css3-gradient-background-set-on-body-doesnt-stretch-but-instead-repeats):

`html {
    height: 100%;
}
body {
    height: 100%;
    margin: 0;
    background-repeat: no-repeat;
    background-attachment: fixed;
}`

Hm...do I need to use browser specific names for the 'background-image' property?

 ----
 **Saving code snippest for later**

 *Date Completed On*
 `<label for="goal">Completed on</label>
      <%= date_field("goal", "date", value: Date.today, max: Date.today) %>`

From the goals partial, removing the 'completed today' list for now:

`<div>
  Completed Today (yay, good work!):
  <br>
  <% completed_goals.each do |goal| %>
    <%= goal.display_name %>
    <br>
  <% end %>
</div>`




