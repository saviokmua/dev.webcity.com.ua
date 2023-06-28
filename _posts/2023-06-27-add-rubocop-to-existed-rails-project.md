---
layout: post
author: savio
title: Add Rubocop to existed Rails project
categories: ["ruby on rails", 'rubocop']
tags: ['ruby', 'ruby on rails', 'rubocop', 'linter', 'style' ]
image:
  path: /assets/img/posts/rubocop-logo.png
  alt: Rspec
---

### Add gems to Gemfile

```ruby
gem 'rubocop', require: false
gem 'rubocop-performance', require: false
gem 'rubocop-rails', require: false
gem 'rubocop-rspec', require: false
```

### config/application.rb

```ruby
class Application < Rails::Application
  config.generators.after_generate do |files|
    parsable_files = files.filter { |file| file.end_with?('.rb') }
    system("bundle exec rubocop -A --fail-level=E #{parsable_files.shelljoin}", exception: true)
  end
  ...
end
```
 
### .rubocom.yml

```yml
require:
  - rubocop-performance
  - rubocop-rspec
  - rubocop-rails

Rails:
  Enabled: true

AllCops:
  NewCops: enable
  Exclude:
    - "db/schema.rb"
    - "Gemfile"
    - "lib/tasks/*.rake"
    - "bin/*"
    - "config/puma.rb"
    - "config/spring.rb"
    - "config/environments/development.rb"
    - "config/environments/production.rb"
    - "spec/spec_helper.rb"

Style/Documentation:
  Enabled: false
```
