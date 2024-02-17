---
layout: post
title:  "How to embed Active Record models in Action Text"
---

Hope your 2024 is off to a good start! I had a restful winter break and I'm excited to be back!

Did you know that we can embed any `ActiveRecord` model in our Rails app as an 'attachment' within rich text content using `ActionText`? 

Let me show you what I mean and how `ActionText` embeds work.

## ActionText Overview

With `ActionText` we can store content with formatting (bold, italics, etc), bullets, links, headers and attachments. `ActionText` uses the Trix editor to create and store the rich text content. `ActionText` also uses `ActiveStorage` for attachments like images and files. 

But in addition to these attachments, `ActionText` has a feature that allows us to embed *any* `ActiveRecord` model in our rich text content. All we need is a signed global id that uniquely identifies our model object. 

## Signed Global Id

A [Global ID](https://github.com/rails/globalid#global-id---reference-models-by-uri) is an app wide URI that uniquely identifies a model instance. A signed global id is a global id that's been signed for added security to ensure that the data hasn't been tampered with.

In order to embed our models into rich text content, the only requirement is that our model class supports signed global Ids and has a `to_sgid()` method. 

All `ActiveRecord` models have a `to_sgid()` method as `ActiveRecord::Base` mixes in the `GlobalID::Identification` concern. 

```ruby
class Message < ApplicationRecord
  has_rich_text :content
end
...
>> message = Message.first
=> #<Message:0x000000010ee79b18

>> message.to_sgid
=> #<SignedGlobalID:0x0000000005d110>

>> message.to_sgid.to_s
=> "eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaEpJaHhuYVdRNkx5OW1hWE5vYVc1bkwwMWxjM05oWjJVdk1RWTZCa1ZVIiwiZXhwIjoiMjAyNC0wMi0xMVQxNDozNjowMS4zOTBaIiwicHVyIjoiZGVmYXVsdCJ9fQ==--44cfdca6b1556abb7872e94491cd929b417dab7b"
```

That string is the signed global id, it has two parts. The part before the `==` is the identifier and the part after is the signature.

We can use this `sgid` to embed the `message` model instance within `ActiveText` content.

## How to Embed ActiveRecord Models

With the `sgid` of a model in hand, we can embed that model into `ActionText` content using the `<action-text-attachment>` element.

```html
<p>Hello, This is a paragraph in a post that uses rich text content using ActionText and here is a Message I'd like to embed: <action-text-attachment sgid="eyJfcmFpbHMiOnsib…"></action-text-attachment>.</p>
```

Now `ActionText` will use the "eyJfcmFpbHMiOnsib..." identifier to find the `Message` object. 

```ruby
>> message_sgid = message.to_sgid
=> #<SignedGlobalID:0x0000000007f8a0>

>> GlobalID::Locator.locate message_sgid
=> #<Message:0x000000010eeb3ac0
```

Next, this part is pretty cool, it will find the default partial `_message.html.erb` under `app/views/messages` and it will render the `html` generated from this partial inside the `<action-text-attachment>` element, which has the matching `sgid`.  

Given this simple message partial

```ruby
<span><%= message.body %></span>
```

the final rich text will look like this

```html
<p>Hello, This is a paragraph in a post that uses rich text content using ActionText and here is a Message I'd like to embed: <span>This is the message body</span>.</p>
```

That's handy. And this is a possible with any old `ActiveRecord` model in our application. This can be used to embed/tag a user or embed product preview or a book review, etc.

There are two other things to know about this `ActionText` embed feature: 

1. `ActionText` uses the existing message partial by default but we can define our own partial to use specifically for when the model is rendered as an attachment. This is done by defining an instance method `to_attachable_partial_path` and a matching partial.
2. We can also define what to render in case of missing attachments. This is done by defining a class method `to_missing_attachable_partial_path` and a partial to render when a model instance is not found using the given `sgid`.

You can find more info in the [`ActionText` guides](https://edgeguides.rubyonrails.org/action_text_overview.html) (which I happen to be reading earlier this week when I came across this feature). I can use something like this in one of my projects, where I store book notes using `ActionText` and want to render a reference to other book model instances.

There is always value in reading the docs :)

That wraps it up. We learned about signed global ids and how to use them to embed any model into any `ActionText` rich text content.
