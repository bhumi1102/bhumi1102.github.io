---
layout: post
title:  "Welcome to Jekyll!"
date:   2017-08-04 16:17:54 -0700
categories: jekyll update
---
Code highlighting

{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}
