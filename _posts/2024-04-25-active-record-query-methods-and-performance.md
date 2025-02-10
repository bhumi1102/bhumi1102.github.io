---
layout: post
title:  "Active Record Query Methods and Peformance"
---

Active Record is great, it makes it easy for us to interact with the database without writing much SQL. But sometimes *too* easy. For example, an innocent statement like this, `User.all.each`, can grind our application to a halt, if we had 10M records in our `users` table. Because it'll try to load all of them into memory. But then again, we likely don't have 10M users. Context matter when comparing performance of different options. 

Let's see this with a few examples - comparing `sort_by` and `order`, comparing `map` and `pluck`, and comparing `length`/`count`/`size` and `exists?`/`present?`/`any?`.

## Sorting: `order` and `sort_by`
Let's look two options for sorting: `sort_by` and `order`. The `sort_by` method is from Ruby's `Enumerable` module, which is a mix-in available on things like Arrays and Hashes. It is independent of Rails. 

When we use `sort_by`, the sorting is done in memory. In the example below, all of the associated `books` records being sorted will need to be loaded into memory.

```ruby
sorted_books = user.books.sort_by { |book| book.title }
```

The `order` method is from Active Record. When we use `order` with an Active Record model, it runs a SQL query with an 'ORDER BY' clause.

```ruby
sorted_books = user.books.order(title: :asc)
```

and the resulting SQL:

```sql
SELECT "books".* FROM "books" WHERE "books"."user_id" = 1 ORDER BY "books"."title" ASC
```

You might conclude that `sort_by` seems expensive and we should always use `order`. Why would we load all the records in memory to do the sorting when we can make the database do the work? In general, yes making the database do the work is preferred. However, if we have a dataset that is *already* loaded in memory (for other reasons), then `sort_by` is actually faster. Context matters when it comes to performance. 

Also, I recently learned that there is a `.loaded?` method on Active Record associations, to check if the association is already loaded, i.e. `user.books.loaded?`.

There is a parallel to this for **filtering and `select` and `where` methods**. The `select` method comes from Ruby and does the filtering in memory. While the `where` method comes from Active Record and runs a SQL query against the database. So same performance considerations.

## Selecting Fields: `pluck` and `map`
Let's say we need all the of titles from the `Book` model. We can get them using `map`:

```ruby
books = current_user.books
books.map(&:title)
```

This will load all of the records in `books` into memory and then iterate over that collection to get only the `title`. But we will have loaded all fields of the `Book` model into memory.

We can use the `pluck` method instead to get the `title` without loading all the fields of an Active Record model into memory just to discard all but one field.

```ruby
books.pluck(:title)
```

This is resulting SQL from the `pluck` method. Note it doesn't do `select *`:

```sql
SELECT "books"."title" FROM "books"
```

So `map` requires loading all the fields for the Active Record models into memory and `pluck` makes a DB query. Which one is better? Actually, `pluck` does not always make a DB query. If the records are *already* loaded into memory, it'll use those. And if not, it will query for only the desired field.

## Counting Records
Next, let's compare the methods `length` , `count`, and `size`.

The method `length` comes from Ruby and is available on enumerables. The `count` method is from Active Record, and `size` method is available on Active Record relations.  

```ruby
>> books_array = Book.all.to_a
>> books_array.length
=> 11

>> books_array.loaded?
=> true

>> Book.count
  Book Count (13.6ms)  SELECT COUNT(*) FROM "books"
=> 11

>> books = current_user.books
>> books.size
=> 5
```

The `length` method will load all records into memory. This is different from `count` and `size`, which generate SQL queries to count the records directly in the database *without* loading all records into memory. And `size` is the best-of-both-worlds, in the sense that if the records are already loaded into memory it will use those and not make a SQL query.

And as a bonus, let's add the methods **`present?`/`exists?`/`any?` for checking whether records are present**, as they parallel the counting methods `length`/`count`/`size`, respectively.

```ruby
>> books.loaded?
=> true

>> books.present?
=> true

>> books.exists?
  Book Exists? (7.6ms)  SELECT 1 AS one FROM "books" LIMIT $1  [["LIMIT", 1]]
=> true

>> books.any?
=> true
```

Hope this illuminates some things about Active Record methods and performance. Because when you look at calls like `books.count` and `books.length`, it isn't immediately obvious which one makes a DB query or not? Or which one is a Ruby method and which one comes from Rails and Active Record.

Also when it comes to performance context matters. When deciding between multiple methods that accomplish the same thing, we have to ask: Will it run a database query? Will it load results into memory? Will it use results already loaded in memory? And what is the scale of your data?

