---
layout: post
title:  "How I Use Action Mailbox to Email Myself Book Notes"
---

This one is about receiving incoming email in your Rails application using Action Mailbox. You've likely used Action Mailer, which is for *sending* emails from your application. Action Mailbox is for *receiving* email.

A long time ago, I used to email myself all kinds of things - a quote I liked, or a book recommendation, a reminder. Nowadays I've set up one of those fancy personal-knowledge-management-second-brain-note-taking systems like a civilized person. (not really! I write everything in markdown in Obsidian, including this newsletter).

I have been fiddling with a book notes app for personal use though. I added this feature to it: when I'm reading/listening to a book, I can email myself a short note with a thought or an idea that occurs to me and have it show up in the app.

![Action Mailbox with reading notes](/assets/images/action_mailbox_reading_notes.png)

Let me show you how I did this and how Action Mailbox works:

## Action Mailbox Overview
Here are some facts to know about Action Mailbox:

- Action Mailbox routes incoming emails to controller-like mailboxes for processing in your Rails application. 
- The incoming emails are routed asynchronously using Active Job. 
- The emails are turned into `InboundEmail` records using Active Record, which can interact directly with the rest of your domain model.
- The original emails are stored via Active Storage while processing along with their status. They're also automatically deleted after some configurable time (called Incineration).
- In order to actually receive email, we configure something called an ingress. Email providers like Mailgun, Postmark, Sendgrid, etc. have a way to configure these. The instructions are in the Action Mailbox guides. I used Postmark and it was easy enough to setup. Action Mailbox ships with built-in ingresses for local testing.

Okay now let's see some code!

## Receiving Book Notes by Email
First, we **install Action Mailbox** with `bin/rails action_mailbox:install`.
This will generate `application_mailbox.rb` and copy over some migrations. 

The new table `action_mailbox_inbound_emails` stores incoming messages and their processing status. The `status` column can be one of `pending`, `processing`, `delivered`, `failed`, or `bounced`.

Next, we **setup routing**. In this case we are matching an email sent to an address like `book-42@whatever.com` to a mailbox we call `active_reading_notes`. The `application_mailbox.rb` looks like this:

```ruby
class ApplicationMailbox < ActionMailbox::Base
  # routing /something/i => :somewhere
  routing(/^book-\w+@/     => :active_reading_notes)
end
```

Now we need to actually **create this mailbox** which does not exist yet. We do this with `bin/rails generate mailbox active_reading_notes`. 

Next, we **implement the `process` method** of the `ActiveReadingNotesMailbox` to do something with the incoming email. This is the interesting part. Here you can create other models based on the email data, you can send emails, you can start a background job. You can do anything you want.

I'm creating an `ActiveReadingNote` based on the body of the email, associated with the `Book` identified by the integer in the `mail.to` field. My `Book` model `has_many :active_reading_notes`.

```ruby
class ActiveReadingNotesMailbox < ApplicationMailbox
  def process
    book_regex = /^book-(\w+)@/i

    # find the book from email address
    email_to = mail.to.first
    book_id = email_to.match(book_regex)[1]
    book = Book.find(book_id)

    body = mail.body.decoded

    book.active_reading_notes.create(content: body)
  end
end
```

(Note: this is a 'naive' implementation, for harding it we would handle if the book doesn't exist and use better identifiers).

Notice that in the mailbox `process` method, we have access to the `mail` object, which is from the [`Mail Library`](https://github.com/mikel/mail?tab=readme-ov-file#usage) and has all the fields that you'd expect in an email.

How do we **test if this is working locally**? Action Mailbox mounts a conductor controller at `http://localhost:3000/rails/conductor/action_mailbox/inbound_emails`. We can use this to create new `InboundEmail` and send it. We can also see an index of all the `InboundEmails` in the system and their state of processing. That's pretty cool.

I'm displaying all my "active reading notes" (i.e. emails I sent to my app while reading) on the book's `show` page, like you see in the screenshot at the top.

That's all for this one. I challenged myself to keep this as short and to the point as possible.

More details about the `mail` object, incineration, and how to configure Action Mailbox with an external email provider (e.g. postmark) in production are in the new and improved [Action Mailbox](https://edgeguides.rubyonrails.org/action_mailbox_basics.html) guide.