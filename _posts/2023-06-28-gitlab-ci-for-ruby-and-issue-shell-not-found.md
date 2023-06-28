---
layout: post
author: savio
title: Gitlab CI for Ruby 3.2.2 and issue "shell not found"
categories: ['gitlab', 'ci']
tags: ['gitlab', 'ci', 'validate', 'yml', 'issue', '3.2.2', 'ruby' ]
image:
  path: /assets/img/posts/gitlab-logo.png
  alt: Rspec
---

When I tried setup CI of Gitlab for Ruby on Rails project based on ruby 3.2.2
```
image: 'ruby:3.2.2'
```
I faced the next issue like
```
Skipping Git submodules setup
Executing "step_script" stage of the job script 00:00
Using docker image sha256:...2fda for ruby:3.2.2 with digest ruby@sha256:54d8 ...
shell not found
Cleaning up project directory and file based variables 00:01
ERROR: Job failed: exit code 1
```

One interesting moment, for ruby version `3.0.5` all works correctly!

My solution is next:
change image name in `.gitlab-ci.yml` to
```
image: 'ruby:3.2.2-slim-buster'
```

and all works correctly!

if you understand, why it not working correctly for image `ruby:3.2.2`
please let me know in the comment. thanks!
