layout: post
title:  "How to Use Enums in Rails Active Record"
---

Suppose you're making a book tracking app to keep a log of all the books you consume. And you want to log the format. As in, was it an ebook, an audiobook or was it an actual physical paper copy? You would create a column named `format` on your model. 

We want it to hold one of 3 values and be able to query it by name as it `where book.format = "ebook"`. The next question is what *type* should this column be? This sounds like an enumeration. 

Rails does support this concept of enumerations. `ActiveRecord::Enum` allows us to declare named attributes whose values map to integers in the database. And these attributes can be queried by name.

**Here's everything you need to know to use Enums in your Rails app**. 

We declare the enums on the model. 

```ruby
class Book < ApplicationRecord
  
  enum format: {
    paper: 0,
    ebook: 1,
    audiobook: 2
  }
  
  enum reading_mode: {
    read: 0,
    skim_skip: 1,
    parse: 2,
    to_be_continued: 3,
    reread: 4
  }
  ...
end
```

In addition to `format`, I also want to track something I call a `reading_mode` as I don't read most books linearly cover to cover like a normal person. 

Here is part of the migration. We define these columns as `integer`.

```ruby
def change
  create_table :books do |t|
    ...
    t.integer :format
    t.integer :reading_mode

    t.timestamps
  end
end
```

With that in place, here is how we create, update and query books with those enums.

To create a book with the enum fields

```ruby
Book.create!(title: "The Hobbit", format: "paper", reading_mode: "read")
```

and to query and update the enum fields

```ruby
> book.format
=> "paper"

> book.paper?
=> true

> book.ebook?
=> false

> book.format = "ebook"
=> "ebook"

> book.format
=> "ebook"

> book.ebook?
=> true
```

I also wanted to get all of the enum values so I can display them in a dropdown in the UI where a user can select the value by name when adding a new book. 

We can get a hash with all keys and values of an enum like this

```ruby
> Book.formats
=> {"paper"=>0, "ebook"=>1, "audiobook"=>2}

> Book.reading_modes
=> {"read"=>0, "skim_skip"=>1, "parse"=>2, "to_be_continued"=>3, "other"=>4}
```

From that hash I can derive an array of just the string values for the user to select from. Rails automatically converts between string and integer when storing enums to the database.

## Using Array vs. Hash for Enum

For the book format and reading_mode I am using a hash to define the enum. We could also use arrays `enum :format, [:paper, :ebook, :audiobook]`. If we use arrays the values in the databases will be 0, 1, 2 respectively for paper, ebook, and audiobook. 

This seems okay *but* it's order dependent. So if we change the order, the semantics will change and our existing logic break. So the recommended way to use Active Record enums is with hashes.

## Adding Prefixes and Suffixes

In case the enum names are not clear when read on their own, we can add a prefix or suffix to indicate what they represent. For example the reading modes might be more clear if written as `reading_mode_parse` instead of just `parse` by itself. We can do this by passing in an option to the enum declaration

```ruby
  enum reading_mode: {
    read: 0,
    skim_skip: 1,
    parse: 2,
    to_be_continued: 3,
    reread: 4
  }, prefix: true
```

That is all there is to Active Record Enums in Rails. 
