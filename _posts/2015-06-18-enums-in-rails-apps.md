---
layout: post
title:  "Enums in Rails Apps"
date:   2015-06-18
categories: mediator feature
tags: code
image: /assets/images/background_image.jpg
---


In March 2014, Rails 4.1 introduced support for an enum type. When I heard the news, I was excited. Previously, this functionality was only possible by using a gem like `simple_enum`. Both options take similar approaches and map the value to integers stored in your database.

*Just in case you're not familiar with the enum type...* An enum (pronounced "e-noom") is a string whose value is limited to a list of permitted values. This situation comes up routinely like:

- vehicle_type: bike, motorcycle, car, spaceship
- review_status: approved, rejected, pending
- marital_status: married, single, divorced

Using enums in Rails 4.1+ requires just two steps:

1. add an integer to the table
2. define an enum attribute in your model (as seen below)

```
class Review < ActiveRecord::Base
  #option 1
  enum status: [:pending, :approved, :rejected]

  #option 2
  enum status: { pending: 0, approved: 1, rejected: 2 }
  #to explicitly map the relation between the attribute and database integer
end
```

By adding the single line above, you gain all of the following functionality:

```
review.pending!    #review.update! status: 0

review.pending?    #=> true

review.status      #=> “pending”

Review.statuses    #=> { "pending" => 0, "approved" => 1, “rejected” => 2 }

Review.pending     #=> Review.where(status: 0)
```

Depending on how many values your attribute can have, you can avoid writing a lot of code. It can't get better than this, right?

#But wait...
As an alternative, you can store an enum type directly in the database. MySQL and PostgreSQL both support this, and we do this often at [BuildZoom](https://www.buildzoom.com).

You can add it to your table with a migration like any other database change:

```
add_column :reviews, :status, "ENUM('pending', 'approved’,’rejected’')", :default => ‘pending’, :after => 'stars'
```

This alternative does offer a key advantage in that an enum type in the database is more convenient and revealing when working with non-engineers. At BuildZoom and many other early-stage startups, non-engineers (product, marketing, and operations) use the data, but can't access the codebase or don’t know how to query the database. In these cases, we can share queries or quickly export data to a spreadsheet or csv and the value of the enum field is clear to everyone.

Going this route does have disadvantages in that you will have to write your own helper methods and an enum field takes up more space than a single byte integer field.

Both options have pro's and con's, but next time you need an enum, consider this alternative.

Links

* [Rails 4](http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html)
- [MySQL](https://dev.mysql.com/doc/refman/5.0/en/enum.html)
- [Thoughtbot blog](https://robots.thoughtbot.com/whats-new-in-edge-rails-active-record-enum)
- [simple_enum gem](https://github.com/lwe/simple_enum)
