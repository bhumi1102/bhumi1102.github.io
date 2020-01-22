---
layout: post
title:  "ENV variables in Rails"
date:   2020-01-22
---

**Environment specific configuration for storing senstitive keys and secrets**

How this is done requres considering the deployment environment. So for heroku specifically:
https://devcenter.heroku.com/articles/config-vars

After reading this and learning about the existence of this gem figaro, I decided to use it.
It's compatible with Heroku config vars [Figaro Repo](https://github.com/laserlemon/figaro)

Using it like this in omniauth.rb initializer

{% highlight ruby %}
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :google_oauth2, ENV["GOOGLE_APP_ID"], ENV["GOOGLE_APP_SECRET"]
  provider :facebook, ENV["FACEBOOK_APP_ID"], ENV["FACEBOOK_APP_SECRET"]
end
{% endhighlight %}

This gem adds a config/applicaiton.yml in .gitignore.  Looks like this:
{% highlight ruby %}
# Add configuration values here, as shown below.
#
# pusher_app_id: "2954"
# pusher_key: 7381a978f7dd7f9a1117
# pusher_secret: abdc3b896a0ffb85d373
# stripe_api_key: sk_test_2J0l093xOyW72XUYJHE4Dv2r
# stripe_publishable_key: pk_test_ro9jV5SNwGb1yYlQfzG17LHK
#
# production:
#   stripe_api_key: sk_live_EeHnL644i6zo4Iyq4v1KdV9H
#   stripe_publishable_key: pk_live_9lcthxpSIHbGwmdO941O1XVU

FACEBOOK_APP_ID: "fake"
FACEBOOK_APP_SECRET: "fake"

GOOGLE_APP_ID: "fake.apps.googleusercontent.com"
GOOGLE_APP_SECRET: "fake-"

production:
  FACEBOOK_APP_ID: "fake"
  FACEBOOK_APP_SECRET: "fake"

  GOOGLE_APP_ID: "fake.apps.googleusercontent.com"
  GOOGLE_APP_SECRET: "fake-"
{% endhighlight %}

