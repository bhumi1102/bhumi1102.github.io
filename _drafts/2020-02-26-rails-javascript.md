---
layout: post
title:  "Rails javascript, asset pipeline, turbolinks, application.js"
date:   2020-02-26
---

Question that triggered the research:
Why is the timezone cookie getting set and working correctly locally but not in production?

`
$(document).on('turbolinks:load', function() {
  //detectChange();
  //this cookie is used in around_action in application_controller
  document.cookie = 'time_zone='+Intl.DateTimeFormat().resolvedOptions().timeZone+';';
  console.log(Intl.DateTimeFormat().resolvedOptions().timeZone);
})`

This is in `dashboard.js` and I see this javascript loaded on the page (in dev tools). But probably not in time to catch the `$(document).ready` or `$(document).on(turbolinks:laod)` type events. (side not: `$(document).on(page:laod)`) doesn't work at all even locally)

Gaps in deep understanding, aka opportunities of thigns to explore:
- How and when is application.js generated from all the other javascript files? read about the asset pipeline.
- What is the difference between the different DOM events? document.ready, document.on. Read MDN documentation. Search for how this relates to rails asset pipeline and order of events.
- 