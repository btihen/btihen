---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Rails 7.0 Using Events"
subtitle: "Using Events for simple Controllers and Loose coupling"
summary: ""
authors: ['btihen']
tags: ['rails', 'packwerk', 'architecture', 'design']
categories: []
date: 2022-04-23T01:20:00+02:00
lastmod: 2022-04-23T01:20:00+02:00
featured: true
draft: true

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## Organizing Code

In Rails its all too easy to accidentally tightly couple controller activities with follow-up actions - resulting in bloated controllers and / or tight coupling with models.

A relatively simple fix to help with this is to use Events.

## Ruby and Rails Environment

Using Rails 7 & Ruby 3.1.2 - I found that it is important to update my ruby environment - so before we start this is what I didn't remove errors:

```bash
# I've had the error several times without updating:
# /Users/btihen/.rbenv/versions/3.1.0/lib/ruby/gems/3.1.0/gems/bundler-2.3.8/lib/bundler/rubygems_ext.rb:18:in `source': uninitialized constant Gem::Source (NameError)
#
#       (defined?(@source) && @source) || Gem::Source::Installed.new
#                                            ^^^^^^^^
# Did you mean?  Gem::SourceList
# this seems to fix it:
# https://bundler.io/guides/bundler_2_upgrade.html
# https://stackoverflow.com/questions/4859600/bundler-throws-uninitialized-constant-gemsilentui-nameerror-error-after-upgr
rbenv local 3.1.2
gem update --system
gem install bundler
gem install rails
rbenv rehash
```

## Rails Project - Simple Blog

Since my other projects are using `esbuild` I use that here too

```ruby
rails new rails_events -T --database=postgresql --css=bootstrap --javascript=esbuild
cd rails_events
bin/rails db:create

# add the helper gem
bundle add ma

```

NOTE: to turn a folder into a package - add the file: `package.yml` in the package folder - this will be described in more detail as we go.

## Configure Packages


## Gems

* wisper_next - https://gitlab.com/kris.leech/wisper_next (active, with sync and async options) - https://github.com/krisleech/wisper (inactive 3 years)
* ma - https://gitlab.com/kris.leech/ma (active, Events as Objects, built upon wisper_next)
* event_bg_bus - https://github.com/indiereign/event_bg_bus (active with backgrounding) - https://github.com/kevinrutherford/event_bus (no activity 8 years)
* https://github.com/RailsEventStore/rails_event_store

## Resources

* https://www.toptal.com/ruby-on-rails/the-publish-subscribe-pattern-on-rails


## Going Further with Events & DDD

* https://github.com/RailsEventStore/rails_event_store
* https://medium.com/kontenainc/event-driven-microservices-with-rabbitmq-and-ruby-7a54ae01b285
* https://www.globalapptesting.com/engineering/design-the-unknown-with-the-help-of-event-storming
* Event-Driven Architecture and Messaging Patterns for Ruby Microservices - Kirill Shevchenko - https://www.youtube.com/watch?v=e9AAUy4kkek
