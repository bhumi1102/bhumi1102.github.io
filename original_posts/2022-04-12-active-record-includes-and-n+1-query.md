layout: post
title:  "Active Record includes and N + 1 Query Problem"
---

Consider the following code

```ruby
books = Book.limit(10)

books.each do |book|
  puts book.author.last_name
end
```

In the above code, a query is made each time the lastname of an author of a book is needed in `puts book.author.last_name`. A single query for each book. This is because of *lazy loading*. Lazy loading is when Rails avoids querying the database until a value is actually needed.

If we have N books, we will end up making N queries to the Database in the `.each`. And, of course, we'll make a single query to get the 10 books in the first line `Book.limit(10)`. Hence we will have made N + 1 queries. 

If you've been around Rails a while you already know this as the "N+1 query problem". This post shows the different options for solving this problem.

We can solve this problem with *eager loading*. Eager loading is about loading the associated records of a model object ahead of time, using as few queries as possible.

We do this with Active Record methods `includes`, `preload`, and `eager_load`.

## includes
We can use `includes` method to load the `author` association like this `Books.includes(:authors).limit(10)`.

That line of code will produce the following 2 SQL queries (instead of 11 queries)

```ruby
SELECT `books`* FROM `books` LIMIT 10

SELECT `authors`.* FROM `authors`
WHERE `authors`.`book_id` IN (1,2,3,4,5,6,7,8,9,10)
```

## includes vs. joins
Using `includes` result in a `left outer join` in the SQL query, while using `joins` results in a `inner join` query. Both `joins` and `includes` support conditions on the query with `where`. 

`Author.includes(:books).where(books: { out_of_print = true})`

In the case with the `includes` if there were no books for the authors, all the authors would be still loaded. If we use `joins` instead, no records will be loaded if there were no books for authors.

It makes sense to use `joins` when we need to specify conditions on the eager loaded associations. Note that using `joins` on its own does not prevent an N + 1 query.

## preload and eager_load
There are two other methods `preload` and `eager_load` to address the N + 1 query problem.

`preload` works similarly to `includes`. It generates the same 2 queries instead of N + 1 queries. Unlike `includes` method, `preload` does not support specifying conditions with `where`. 

I personally like `preload` as the method name is clear about what we are doing compare to `includes`. And in general using `joins` is more clear when we need to specify conditions.

The other option is the `eager_load` method, Active Record forces eager loading by using `left outer join` for all specified associations. This would result in 2 queries instead of N + 1, with both queries using `left outer join`.

In general, we can be on the look out for N + 1 query issue whenever we are iterating over a list of records. If we know that we will reference associated models for each record, we can use `includes`, `preload`, or `eager_load` in our original query to load the associated models in one go.

Lazy loading is still useful and sometimes that's exactly what we want. We don't want to load everything up front in one go if we don't actually need it. However, when we know we *will* need each author's last name, it makes sense to load it upfront and not query each one by one.
