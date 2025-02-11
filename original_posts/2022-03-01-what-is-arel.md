layout: post
title:  "What is Arel: Between ActiveRecord and SQL Queries"
---

I recently learned that ActiveRecord uses Arel under the hood. What *is* Arel? When would it be useful in practice? let's find out!

### Motivation

ActiveRecord is useful and its DSL allows us to write many queries with `.find_by` and `.where`. But it does have limitations in the types of SQL queries we can compose in the `where` clause. 

It can combine statements using AND and comparison operators like = and !=. But it doesn’t provide a DSL for numeric comparisons like <= or >= for example. In that case, Rails developers usually write out a SQL string literal. Using Arel could be another (better?) way to this. 

### What is Arel

Arel is a library used for *constructing* SQL queries. Introduced in Rails 3. Arel stands for "A relational Algrebra" from [Arel](https://github.com/rails/arel). It used to be a separate library (before [2017](https://github.com/rails/arel/blob/master/History.txt)) but is now bundled with `ActiveRecord`. Here is the [source](https://github.com/rails/rails/blob/main/activerecord/lib/arel/table.rb), it's a quick read if you're curious.

Arel *gives us access* to SQL AST (Abstract Syntax Tree) for our model. That's really cool because we can systematically build powerful queries that are readable. 

Okay enough preamble, let's see some code.

## How does Arel work?

Every ActiveRecord model has a `arel_table` method to access the underlying Arel interface. ActiveRecord `where` uses Arel under the hood. 

Assuming `my_table = User.arel_table`, this

```ruby
User.where(my_table[:id].in([1,2,3])).to_sql
```

is the same as

```ruby
User.where(id: [1,2,3]).to_sql
```

They both return the same SQL string. Try it in your rails console!

```ruby
> User.where(my_table[:id].in([1,2,3])).to_sql
=> "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"id\" IN (1, 2, 3)"
> User.where(id: [1,2,3]).to_sql
=> "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"id\" IN (1, 2, 3)"
> 
```

The above `in` is one of the predicate methods that can be used. We can get list of all predications by calling `Arel::Predications.instance_methods`. Here is a subset from my console:

```ruby
> Arel::Predications.instance_methods
=> [:does_not_match_regexp,
 :does_not_match_any,
 :does_not_match_all,
 :gteq_any,
 :gteq_all,
 :gt_any,
 :gt_all,
 :lt_any,
 :in,
 :lt_all,
 :lteq_any,
 :lteq_all,
 ...
 :not_between,
 :gt,
 :not_in_any,
 :not_in_all,
 :matches_regexp,
 :matches_any,
 :matches_all,
 :does_not_match]
```

One nice thing about Arel is that it doesn't touch the database, until we pass it in as a parameter to the `where` clause. This means we can construct custom queries even if the model doesn't exit by defining our own Arel table.

Here is an example of a query built that way with `Arel::Table.new`.

```ruby
veg = Arel::Table.new(:vegetables)
query = veg[:created_at].gteq( 5.days.ago ).and(
  veg[:color].eq("green").or(
  veg[:gardener].eq("Emily")
  )
)

> query.to_sql
=> "\"vegetables\".\"created_at\" >= '2022-03-06 22:59:31.928634' AND (\"vegetables\".\"color\" = 'green' OR \"vegetables\".\"gardener\" = 'Emily')"
```

And here are some more SQL AST methods found in `Arel::SelectManager` including `join`, `outer_join` and such.

```ruby
> Arel::SelectManager.instance_methods - Object.methods
=> [:source,
 :limit=,
 :offset=,
 :offset,
 :distinct,
 ...
 :union,
 :locked,
 :group,
 :orders,
 :join_sources,
 :outer_join,
 :join,
 :project,
 :lock,
 :from,
 :order,
 :froms,
 :window,
 :take,
 :having,
 ...]
```

Those are a lot of predicates and lot of AST methods, we could likely do everything imaginable in SQL using those.

Also btw, since Arel is a private API, we should be cautioned that using it has a cost of potential breaking changes when we upgrade Rails. 

So this Arel interface seems useful and kinda cool to see how ActiveRecord  builds the underlying SQL queries. And I like how readable and composable that vegetables query is above.

**Read More**
This post synthesizes information from the below sources:
[this post from 2016](https://www.cloudbees.com/blog/creating-advanced-active-record-db-queries-arel), [this post from 2014](https://thoughtbot.com/blog/using-arel-to-compose-sql-queries), and the [source code](https://github.com/rails/rails/blob/main/activerecord/lib/arel/table.rb)

If you want to dig into Arel some more, here more resources:
[Railsconf 2014 Advanced aRel talk](https://www.youtube.com/watch?v=ShPAxNcLm3o)
[Arel helper Gem](https://www.youtube.com/watch?v=ShPAxNcLm3o)
[Tool to convert SQL queries to Arel](https://www.youtube.com/watch?v=ShPAxNcLm3o)
