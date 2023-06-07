---
layout: post
author: savio
title: Deploy Ruby on Rails project(rvm/capistrano/puma/sidekiq)
categories: ["deploy", "ruby on rails"]
tags: ['ruby', 'ruby on rails', 'deploy', 'ngnix', 'puma', 'capistrano', 'ssh', 'redis', 'rvm', 'sidekiq' ]
image:
  path: /assets/img/posts/this-is-deploy.jpg
  alt: Deploy Ruby on Rails to linux server
---

Let's look at an example when we have a project on Ruby with the next stack:

```
1) Ruby 3.2.2
2) Ruby on Rails 7.0.5
3) Postgres
4) Redis
5) Sidekiq
```

and we want to deploy to our VPS(Ubuntu) server:

```
1) Linux Ubuntu server
2) Nginx
3) Puma
4) Monit
5) Redis
```

# Ruby on Rails Application
![Ruby on Rails](/assets/img/posts/rails-logo.png)

So, let’s create a new Rails project

```
rails new ruby-on-rails-deploy-to-vps
```

in my case it was

```
ruby 3.2.2
Rails 7.0.5
```

Add next gems to Gemfile

```
gem 'pg'
gem 'dotenv-rails'
gem 'sidekiq'

gem 'capistrano', require: false
gem 'capistrano-rvm', require: false
gem 'capistrano-rails', require: false
gem 'capistrano-bundler', require: false
gem 'capistrano-linked-files'
gem 'capistrano3-puma', require: false
```
 in my pet project, I also added
```
gem 'ed25519'
gem 'bcrypt_pbkdf'  
```
it was fixed some issues regarding assets...

and install gems
```bash
bundle install
```

Next, let’s use Capistrano to create the necessary config files.

```
cap install
```

## Capfile

```ruby
# frozen_string_literal: true
    
# Load DSL and set up stages
require 'capistrano/setup'
    
# Include default deployment tasks
require 'capistrano/deploy'

require 'capistrano/scm/git'
install_plugin Capistrano::SCM::Git
    
# Load custom tasks from `lib/capistrano/tasks` if you have any defined
    
require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rvm'
require 'capistrano/puma'
install_plugin Capistrano::Puma  # Default puma tasks
install_plugin Capistrano::Puma::Systemd
    
require 'capistrano/linked_files'
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
```

## config/deploy.rb

```ruby
# frozen_string_literal: true

# config valid for current version and patch releases of Capistrano
lock '~> 3.17.2'

set :application,     'application'
set :repo_url,        'git@github.com:savio.km.ua/XXX.git'
set :user,            'deployer'
set :puma_threads,    [4, 16]
set :puma_workers,    0
set :linked_files, %w[config/database.yml]

# Don't change these unless you know what you're doing
set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/apps/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.error.log"
set :puma_error_log,  "#{release_path}/log/puma.access.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w[~/.ssh/id_rsa.pub] }
# set :ssh_options,     { forward_agent: true}
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true # Change to false when not using ActiveRecord
set :branch, :deploy
set :puma_enable_socket_service, true
set :puma_user, fetch(:user)
set :puma_role, :web
set :puma_service_unit_env_files, []
set :puma_service_unit_env_vars, []
set :puma_enable_socket_service, true

## Defaults:
# set :scm,           :git
# set :branch,        :master
# set :format,        :pretty
set :log_level,     :debug
set :keep_releases, 5

namespace :puma do
  desc 'Create Directories for Puma Pids and Socket'
  task :make_dirs do
    on roles(:app) do
      execute "mkdir #{shared_path}/tmp/sockets -p"
      execute "mkdir #{shared_path}/tmp/pids -p"
    end
  end

  # before :deploy#, :make_dirs
end

namespace :deploy do
  desc 'Make sure local git is in sync with remote.'
  task :check_revision do
    on roles(:app) do
      if `git rev-parse HEAD` != `git rev-parse origin/#{fetch(:branch)}`
        Rails.logger.debug { "WARNING: HEAD is not the same as origin/#{fetch(:branch)}" }
        Rails.logger.debug 'Run `git push` to sync changes.'
      end
    end
  end

  desc 'Initial Deploy'
  task :initial do
    on roles(:app) do
      before 'deploy:restart', 'puma:start'
      invoke 'deploy'
    end
  end

  before :starting,     :check_revision
  after  :finishing,    :compile_assets
  after  :finishing,    :cleanup
  after  :finishing,    :restart
end
```

## config/deploy/production.rb

```ruby
server '<ip address>', port: <ssh port>, roles: %i[web app db], primary: true
set :branch,          'main'
set :rails_env,       :production
set :sidekiq_env,     :production
```

## config/secrets.yml

```
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

generate secret key

```bash
bundle exec rake secret
```

and add to `.env.production` like next

```
SECRET_KEY_BASE: 812be95ecfefb248fa5f41d1defdea27fa350c5a742ca50745b9c4b0d494df1e1a934a904eabacf82b5a7db3d006d9c0ee15dd42fb876bffab72b3013bdf8402
```

copy your ssh public key to the VPS server

```bash
ssh-copy-id deployer@<deploy-server.hostname>
```

## Server
![Ubuntu Server](/assets/img/posts/ubuntu-server-logo.png)

create new user for our app 

```bash
adduser deployer
```

add add user to `sudo`(`/etc/sudoers`) without password

```
deployer ALL=(ALL:ALL) NOPASSWD:ALL
```

Install depended soft

```bash
sudo apt update
sudo apt upgrade
sudo apt install curl libcurl4 libcurl4-openssl-dev nodejs postgresql postgresql-contrib
sudo apt install libpq-dev git mc imagemagick libmagic-dev gnupg2 nginx monit
```

sign in by user deployer

generate ssh keys

```bash
ssh-keygen
```

and add ssh public key to github

```bash
cat ~/.ssh/id_rsa.pub 
```

Remove default nginx host

```
sudo rm /etc/nginx/sites-enabled/default
```

## Redis
![Redis server](/assets/img/posts/redis-logo.png)

[install redis server](https://redis.io/docs/getting-started/installation/install-redis-on-linux/){:target="_blank"}

## Rvm
![RVM](/assets/img/posts/rvm-logo.png)

[install RVM](https://rvm.io/rvm/install)
and Ruby (for current example it is version 3.2.2)

```bash
source ~/.bashrc
rvm install 3.2.2
rvm --default use 3.2.2
```

## Postgresql 
![Postgresql](/assets/img/posts/pg-logo.png)
create DB and user

```sql
sudo -u postgres psql
create user deployer with password 'password';
alter role deployer superuser createrole createdb replication;
create database application_production owner deployer;
\q
```

## Monit
![Monit](/assets/img/posts/monit-logo.png)

uncomment in `/etc/monit/monitrc` the next linas:
```
 set httpd port 2812 and
     use address localhost
     allow localhost
```


## Deploy
![Capistrano](/assets/img/posts/capistrano-logo.png)
```bash
bundle exec cap production puma:systemd:config
bundle exec cap production puma:nginx_config
bundle exec cap production sidekiq:install
bundle exec cap production sidekiq:monit:monitor
bundle exec cap production deploy
bundle exec cap production linked_files:upload_files
bundle exec cap production deploy
```

[Here](https://github.com/saviokmua/deploy-ruby-on-rails-rvm-puma-sidekiq-nginx){:target="_blank"} is an example project with the above description
