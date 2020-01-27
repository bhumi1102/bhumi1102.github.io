---
layout: post
title:  "Should I Move to Digital Ocean"
date:   2020-01-27
---

**Current Requiremnt**

**Things I don't know yet / haven't done before**
Deploy rails app to digital ocean - find instruction for this and follow in timebox way
Setup custom domain - find instrucitons and read
Setup SSL - find instruciton and determine if *free* is possible

Seems like free for DNS management and SSL Certificates is possible from [this](https://www.digitalocean.com/docs/networking/dns/)

**Here is what deploying Rails App instructions look like (from July 2015)**
[Deploy Rails App with Git Hook son Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-rails-app-with-git-hooks-on-ubuntu-14-04)

This involves: installing `PostgreSQL`, creating a DB user, configuring database.yml, Installing and configurating `Puma`, Installing and configuring `Nginx`, setting up Git remotes, and a 'post-receive' hook script that runs the commands to actually deploy on the server (bundle install, crate db, migrate db, precompile assets, restart puma, restart nginx)

**Another way to deploy Rails app - create a droplet**

This seems better, but seems to be using Rails 4, so keeps these versions updated..hm. [droplet isntructions](https://marketplace.digitalocean.com/apps/ruby-on-rails)

**Are Custom Domains and SSL actually free?**
[billing page](https://www.digitalocean.com/docs/accounts/billing/)
[pricing](https://www.digitalocean.com/pricing/)
This says that DNS Management is one of the things included in all plans. This implies SSL by Let's Encrypt as well, according to other doc I read above.

The things that is not free is databases thought, PostgreSQL the cheapest one is $15/month. So does this mean I'll have to pay this for my Rails droplet?

Yes SSL and Domain DNS management is free. What is not free is creating a droplet to host a Rails APP. Starts at $5/month and the recoomended one was $40/month (they hide the cheaper plans but there is one from $5, $10, $15, etc. I think others like Mubs and StephSmith have mentioned that these were enough for thier web apps and blogging needs respecitively).

Decision - I go with Heroku for my habit tracker app. Will try out DO to compare for the next app.