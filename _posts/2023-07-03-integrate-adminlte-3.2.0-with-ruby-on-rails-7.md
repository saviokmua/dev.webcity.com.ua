---
layout: post
author: savio
title: Integrate AdminLte 3.2.0 on Ruby on Rails 7
categories: ['ruby', 'ruby on rails', 'templates']
tags: ['gems', 'templates', 'ui', 'bootstrap', 'yarn']
image:
  path: /assets/img/posts/adminlte-logo.jpg
  alt: AdminLTE
---

[AdminLTE 3.2.0](https://github.com/ColorlibHQ/AdminLTE){:target="_blank"} responsive administration template based on
- [Boostrap 4.6](https://getbootstrap.com/docs/4.6/getting-started/introduction/){:target="_blank"}
- JS/jQuery plugin


### create new Ruby on Rails project

```bash
rails new adminlte-rails -j esbuild -c bootstrap
```

### add foreman gem

```
gem 'foreman', github: 'ddollar/foreman'
```
after that we can start our project like
```
./bin/dev
```

### add home controller 

```
rails generate controller home index
```
and add changes to `config/routes.rb`

```ruby
root 'home#index'
```

### add admin-lte, jquery and bootstrap packages

```bash
yarn add admin-lte@^3.2.0 jquery
yarn remove bootstrap
yarn add bootstrap@4.6.2
```

### create base admin-lte template

copy `body` tag from `node_modules/admin-lte/starter.html`
to app/view/layouts/application.html.erb

### add css
add below line to `app/assets/stylesheets/application.bootstrap.scss` 
```scss
@import 'dist/css/adminlte.css';
```

### add admin-lte images to project

add below line to `app/assets/config/manifest.js`:
```
//= link_tree ../../../node_modules/admin-lte/dist/img
```
and change tags
```html
<img src="dist/img/AdminLTELogo.png" alt="AdminLTE Logo" class="brand-image img-circle elevation-3" style="opacity: .8">
```
to
```erbruby
<%= image_tag "dist/img/AdminLTELogo.png",  alt: "AdminLTE Logo", class: "brand-image img-circle elevation-3", style: "opacity: .8" %>
```
### added fontawesome(cdn)

go to [https://cdnjs.com/libraries/font-awesome](https://cdnjs.com/libraries/font-awesome){:target="_blank"}

copy and insert to layout like next:
```
<head>
    ...
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" integrity="sha512-iecdLmaskl7CVkqkXNQ/ZH/XLlvWZOJyj7Yy7tcenmpD1ypASozpmT/E0iPtmFIB46ZmdtAc9eNBvH0H/ZpiBw==" crossorigin="anonymous" referrerpolicy="no-referrer" />
    ...
</head>
```

### add jQuery and other js
create file `app/javascript/add_jquery.js` with next content:
```javascript
import jquery from 'jquery'
window.jQuery = jquery
window.$ = jquery
```
and add below lines to `app/javascript/application.js` 
```javascript
import './add_jquery'
import 'admin-lte'
```
and add below lines to `config/initializers/assets.rb`
```ruby
Rails.application.config.assets.paths << Rails.root.join("node_modules/admin-lte/")
Rails.application.config.assets.precompile += %w[dist/css/* dist/js/*.js]
```

[Here](https://github.com/saviokmua/adminlte-with-ruby-on-rails){:target="_blank"} you can see example project with all changes


![admin lte](/assets/img/posts/Adminlte.gif)
