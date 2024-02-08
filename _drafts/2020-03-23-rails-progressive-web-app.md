---
layout: post
title:  "Rails Progessive Web App and Service Worker"
date:   2020-03-23
---

**Service Worker for offlne**

https://onrails.blog/2019/01/08/easy-pwas-the-rails-way/ - some info about how to get service worker and manifest.json out of the asset pipeline / webpacker (one thing I from the rossta person who wrote the serviceworker-rails gem which has some holes)

Service Worker Life Cycle - https://slides.com/askudhay/pwa-is-the-future#/14

Use Google Lighthouse to test my PWA - https://github.com/GoogleChrome/lighthouse

To try next: put this code at the bottom of applicaiton.js (and undo the current changes)
if (navigator.serviceWorker) {
  navigator.serviceWorker.register('/service-worker.js', { scope: './' })
    .then(function(reg) {
      console.log('[Companion]', 'Service worker registered!');
      console.log(reg);
    });
}

This is what the serviceworker-rails gem added/updated, for reference:
`Bhumis-MacBook-Pro:mtf_tracker bhumi$ bundle exec rails g serviceworker:install
Running via Spring preloader in process 25331
      create  app/assets/javascripts/manifest.json.erb
      create  app/assets/javascripts/serviceworker.js.erb
      create  app/assets/javascripts/serviceworker-companion.js
      create  config/initializers/serviceworker.rb
      append  app/assets/javascripts/application.js
      append  config/initializers/assets.rb
      insert  app/views/layouts/application.html.erb
      create  public/offline.html`


**Add to Home Screen (A2HS)**
This was simple once I got the right fields in the manifest file. I put manifest.json in public folder manually. Added a link to in the <head> section in my html and I was able to 'download' my webapp on my own using the browser (have to make sure I navigate to `https`):
  https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Add_to_home_screen