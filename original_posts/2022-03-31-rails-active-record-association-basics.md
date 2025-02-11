layout: post
title:  "Rails Active Record Association Basics"
---

In Rails, an *association* is a connection between two Active Record models. With associations we get convenient utility methods to easily query and do operations on those models.

Rails has the following associations `belongs_to`, `has_many`, `has_one`,  `has_one :through`, `has_many :through`, `has_and_belongs_to_many`

Active Record maintains *primary key* and *foreign key* information between instance of the two models that are related by one of the above associations. But our code doesn't explicitly have to worry about these keys in most cases.

Let's look at each association one by one and see how to declare them in the model class and what the migrations look like. Here are the links for a quick reference:

<a href="#belongs_to">belongs_to</a>
<a href="#has_many">has_many</a>
<a href="#has_one">has_one</a>
<a href="#when_to_use_has_one_vs_belongs_to">When to use has_one vs. belongs_to</a>
<a href="#has_one_through">has_one :through</a>
<a href="#has_many_through">has_many :through</a>
<a href="#has_and_belongs_to_many">has_and_belongs_to_many</a>


## <div id="types_of_associations">Types of Associations</div>

We'll start with the one-to-one associations, then cover one-to-many and many-to-many.

### <div id="belongs_to">belongs_to</div>
Let's start with an example:

```ruby
class Car < ApplicationRecord
  belongs_to :owner
end
```

With `belongs_to`, each instance of the declaring model `Car` is connected to or "belongs to" one instance of the other model `Owner`. 

This means that each `Car` record will have a `owner_id` column that will point to the `id` column of an `Owner` record.

At the migration level, it looks like this

```ruby
create_table :cars do |t|
  t.belongs_to :owner, foreign_key: true
  # ...
end
```

Adding the `foreign_key: true` is not required but doing so adds a database level foreign key constraint.

All this allows us to write `@car.owner` in our code to get the `Owner` object pointed to by a `@car`.

So far, the relationship is one directional. The `@car` knows about its owner, but the owners do not know about their cars. 

We can change that with `has_many`. 

### <div id="has_many">has_many</div>
Continuing the above example, let's say the goal is to be able write `@owner.cars` and get a list of cars that belong to a given owner. Here is how we can do that

```ruby
class Owner < ApplicationRecord
  has_many :cars
end
```

Now we can write `@owner.cars` and it will work. We can also create a new car belonging to a given owner with `@new_car = @owner.cars.create(...)`

Note that in `has_many :cars`, cars is plural. And also note that for `belongs_to :owner` the model name needs to be singular. This is one-to-many relatiionship.

### <div id="has_one">has_one</div>
`has_one` is similar to `has_many` but is used create a bi-directional relationship on the "other side" of `belongs_to` when we know that there is only one instance of the other model that has a reference to this model. 

This creates a one-to-one relationship, while `has_many` creates a one-to-many relationship.

```ruby
class Supplier < ApplicationRecord
  has_one :account
end
```

This implies that we want to model suppliers in our applications such that they have only one account. We can access this by `@supplier.account`

In this case the `Account` would have a `supplier_id` column that is a foreign key to `id` column in `Supplier` model.

The migration would look like this, it adds a unique index as well as and a foreign key constraint at the database level

```ruby
create_table :accounts do |t|
  t.belongs_to :supplier, index: { unique: true }, foreign_key: true
  # ...
end
```

If you noticed that the migration actually uses `belongs_to` on the `Account` while the association is defined as `has_one` on the `Supplier` class, yes that is a bit confusing. 

So, let's clear up when to use `has_one` vs. `belongs_to`.

### <div id="when_to_use_has_one_vs_belongs_to">When to use has_one vs. belongs_to for a one-to-one relationship</div>

Deciding which association to use comes from thinking about the meaning of the data in the real world. In the account vs. supplier case, it makes sense for 'a supplier to own an account' but it makes less sense to say 'an account owns a supplier' I guess. 

Side note: there is an concept known as Domain Driven Design (and a book by the same name) that advocates for matching the data model - the names of tables and their relationships - as closely as possible to the real world names used in that domain. This makes it easier to query and work with the model in our product but also makes it easier to *communicate* about the design to all functions in a team. It give a common language between product managers and developers for example (e.g. when I worked for a healthtech startup, we used terms like 'claims', 'deductible', 'co-insurance' in our application). Anyway, data modeling is a science and an art. 

Back to `has_one` vs. `belongs_to`. One thing we can keep in mind is that the foreign key is always on the `belongs_to` side. So if an account belongs to a supplier, account has a foreign key `supplier_id` pointing to a supplier.

The migration for the `Account` table can be either:
```ruby
create_table :accounts do |t|
      t.bigint  :supplier_id
      # ...
end
```

or we can say `t.references :supplier` or `t.belongs_to :supplier`. They mean the same thing.

Let's look at one more flavor of one-to-one relationship with `has_one :through`. And then we'll move on to many-to-many associations.

### <div id="has_one_through">has_one :through</div>

`has_one :through` association connects the declaring model to one instance of another model by way of a third model. That's a bit abstract, let's see an example:

```ruby
class Customer < ApplicationRecord
  has_one :account
  has_one :account_attribute, through: :account
end

class Account < ApplicationRecord
  belongs_to :customer
  has_one :account_attribute
end

class AccountAttribute < ApplicationRecord
  belongs_to :account
end
```

All this is saying is that we have a `Customer` model that is associated with one `Account`. There are attributes for each account that we store in separate model called `AccountAttribute` for  denormalization. Now if the `Customer`  needs to refer to `AccountAttribute` it needs to go through the associated `Account`. Since `account_attribute` table will have an `account_id` foreign key and `account` table will have a `customer_id` foreign key. We can say `@customer.account_attribute`

The migration for those tables would look like this

```ruby
class CreateCustomerAccountAttributes < ActiveRecord::Migration[7.0]
  def change
    create_table :customers do |t|
      t.string :name
      # ...
    end

    create_table :accounts do |t|
      t.belongs_to :customer
      # ...
    end

    create_table :account_attributes do |t|
      t.belongs_to :account
      # ...
    end
  end
end
```

So `has_one :through` association is for when you have to traverse more than one model to get at the one-to-one relationship we want to express.

Next, let's move on to many-to-many associations.

### <div id="has_many_through">has_many :through</div>

A `has_many :through` association sets up a many-to-many connection with another model *through* a third model.

For example, for a book tracking app that keeps track of which books a user has read over time, we may have a `Book` and a `Reader` model. And a many-to-many relationship called `Readings` that tracks all the books read by a given reader and all the readers of a given book. 

For this *readings* relationship, we may want to store additional fields on the relationship itself. For example, in addition to `book_id` and `reader_id`, we may want to store *when* the user read the book in a field called `start_date` and `end_date`. 

The models would look like this

```ruby
class Reader < ApplicationRecord
  has_many :readings
  has_many :books, through: :readings
end

class Readings < ApplicationRecord
  belongs_to :reader
  belongs_to :book
end

class Book < ApplicationRecord
  has_many :readings
  has_many :readers, through: :readings
end
```

And notice the additional field on `Readings` in the migration

```ruby
class CreateReadings < ActiveRecord::Migration[7.0]
  def change
    create_table :readers do |t|
      t.string :name
      t.timestamps
    end

    create_table :books do |t|
      t.string :name
      t.timestamps
    end

    create_table :readings do |t|
      t.belongs_to :reader
      t.belongs_to :book
      t.datetime :start_date
      t.datetime :end_date
      t.timestamps
    end
  end
end

```


### <div id="has_and_belongs_to_many">has_and_belongs_to_many</div>

`has_and_belongs_to_many` creates a direct many-to-many connection with another model. There is no intermediate model to go through (though we do need to create a join table in the migration, see below). 

Here is an example of the code and migration for `Car` and `Part`, where a car can have many parts and a part can be in many cars.

```ruby
class Car < ApplicationRecord
  has_and_belongs_to_many :parts
end

class Part < ApplicationRecord
  has_and_belongs_to_many :cars
end
```

The migration

```ruby
class CreateCarsAndParts < ActiveRecord::Migration[7.0]
  def change
    create_table :cars do |t|
      t.string :name
      t.timestamps
    end

    create_table :parts do |t|
      t.string :part_number
      t.timestamps
    end

    create_table :cars_parts, id: false do |t|
      t.belongs_to :car
      t.belongs_to :part
    end
  end
end

```

This association makes sense when we know that there are no fields we need to store *on the relationship* itself. This depends on the real world meaning of the relationship and domain of the models. 

That is the main difference between `has_and_belongs_to_many` and `has_many :through`. In terms of which to choose in practice, I've found that most of the time `has_many :through` makes more sense as it gives us an option to store fields on the relationship.

One last note, we are responsible for maintaining our database schema to match our associations. In practice, this means creating join table needed for `has_and_belongs_to_many` associations.
