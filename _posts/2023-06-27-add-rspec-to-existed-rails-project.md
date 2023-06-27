---
layout: post
author: savio
title: Add rspec to existed Rails project
categories: ["ruby on rails", 'rspec']
tags: ['ruby', 'ruby on rails', 'rspec', 'tests', 'db', 'cleaner' ]
image:
  path: /assets/img/posts/rspec-logo.png
  alt: Rspec
---

### Delete existed test folder
Please remove `/app/test` folder

### Add gems to Gemfile

```ruby
group :development, :test do
  gem 'byebug', platform: :mri
  gem 'rspec-rails'
  gem 'factory_bot_rails'
  gem 'faker'
  gem 'vcr'
end

group :test do
  gem 'database_cleaner-active_record'
  gem 'shoulda-matchers'
  gem 'webmock'
end
```

### Run Rspec Generator

```
rails generate rspec:install
```
 
it creates `spec/spec_helper.rb`, `spec/rspec_helper.rb` and `app/.rspec`

### configure spec/spec_helper.rb

```ruby
RSpec.configure do |config|
  config.before(:suite) do
    DatabaseCleaner.strategy = :deletion
    DatabaseCleaner.clean_with(:truncation)
  end

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end
end
```
