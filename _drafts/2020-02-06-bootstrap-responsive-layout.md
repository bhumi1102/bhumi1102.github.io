---
layout: post
title:  "Bootstrap - Making my Grid Layout Responsive"
date:   2020-02-06
---

**Goal - make habit tracker better usable on mobile**

Qeustion: I want the Habit Grid to go underneath the Track Habits on phones (my first 2 users both run into this immediately)

Solution: I made this class name change and 'it works'.  I only superficially understand these naming conventions:
`-    <div class="col-4">
+    <div class="col-lg-4">`

  and 
`-               <div class="col-8">
+               <div class="col-lg-8">`

PULL - experiment with the names in a jsfiddle type place and develop an intuitive understanding. I can use this code snippet to start off [SO](https://stackoverflow.com/questions/40696823/how-to-convert-bootstrap-rows-to-columns-by-viewport-size)

`<section class="container">
  <div class="row">
    
    
    <div class="col-sm-4 col-lg-12">
      <div class="box"></div>
    </div>
    
    <div class="col-sm-4 col-lg-12">
      <div class="box"></div>
    </div>
    
    <div class="col-sm-4 col-lg-12">
      <div class="box"></div>
    </div>

  </div>
</section>`