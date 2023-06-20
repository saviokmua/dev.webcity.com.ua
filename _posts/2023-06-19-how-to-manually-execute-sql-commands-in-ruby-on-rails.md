---
layout: post
author: savio
title: How to manually execute SQL commands in Ruby on Rails
categories: ["ruby on rails"]
tags: ['ruby', 'ruby on rails', 'active record', 'sql' ]
image:
  path: /assets/img/posts/ruby-query.webp
  alt: Ruby Sql query
---


We can use [execute](https://apidock.com/rails/ActiveRecord/ConnectionAdapters/DatabaseStatements/execute){:target="_blank"} method

Example:

```ruby
res = ActiveRecord::Base.connection.execute('select id FROM users WHERE name IS NOT NULL')
res.pluck('id')
=> [1,2,43,60]
```

with params

```ruby
query = "SELECT id, name FROM users WHERE company_id = :company_id" 
res = ActiveRecord::Base.connection.execute(
  ApplicationRecord.sanitize_sql([query, {company_id: 1}])
)
res.to_a
=> [{"id" => 1, "name" => "name"}]
```
