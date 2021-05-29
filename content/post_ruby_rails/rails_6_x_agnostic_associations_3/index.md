---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Rails 6.x - Framework Agnostic Associations - part 2"
subtitle: "Aggregating different, but related Data models (Rails STI alternative)"
summary: "Framework Agnostic Associations - Data models that work across many frameworks"
authors: ["btihen"]
tags: ['Rails', 'Databases', 'Data models', 'Framework Agnostic', 'belongs_to', 'has_one']
categories: []
date: 2021-05-29T01:57:00+02:00
lastmod: 2021-05-29T01:57:00+02:00
featured: true
draft: false

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
## Purpose

In the interest of coding Rails in a way to work well with other code bases, I looking at ways to do complex database relations in a framework agnostic way.  In particular, this article will primarily explore Polymorphic Relationships.

This is the second article in the series.  This article builds on (part 1)[post_ruby_rails/rails_6_x_agnostic_associations_1/]

## Overview

In this case, I want to model a contact list of businesses and people.  Some people will be associated with a company.  Additionally, we will track transactions with each person and business.

The basic model will then look something like:

              ┌───────────┐                         ┌───────────┐
              │           │╲                       ╱│           │
              │  Contact  │─────────────────────────│UserContact│
              │           │╱                       ╲│           │
              └───────────┘                         └───────────┘
                    ┼                                    ╲│╱
                    │                                     │
      ┌─────────────┴───────────┐                         │
      │                         │                         │
     ╱│╲                       ╱│╲                       ╱│╲
┌───────────┐             ┌───────────┐             ┌───────────┐
│           │╲            │           │             │           │
│ Business  │─○──────────┼│  Person   │             │   User    │
│           │╱            │           │             │           │
└───────────┘             └───────────┘             └───────────┘
      ┼                         ┼                         ┼
      │                         │                         │
      └────────────┬────────────┘                         │
                   │                                      │
                  ╱│╲                                     │
             ┌───────────┐                                │
             │           │╲                               │
             │  Remark   │─○──────────────────────────────┘
             │           │╱
             └───────────┘

                        Created with Monodraw

                  *virtual attribute

                 Created with Monodraw

## Rails app and first Models

    ┌────────────┐             ┌───────────┐
    │            │╲          1 │           │
    │  Business  │─○──────────┼│  Person   │
    │-legal_name │╱0..*        │-full_name │
    └────────────┘             └───────────┘

We discussed/explained in (part 1)[post_ruby_rails/rails_6_x_agnostic_associations_1/]

## Polymorphic (STI) - sometime called inverse polymorphic


                   ┌─────────────┐
                   │   Contact   │
                   │  relations* │
                   │+display_name│
                   └─────────────┘
                          ┼
                          │
          ┌───────────────┴────────────┐
          │                            │
         ╱│╲                          ╱│╲
    ┌─────────────┐             ┌─────────────┐
    │  Business   │╲            │    Person   │
    │ -legal_name │─○──────────┼│ -full_name  │
    │+display_name│╱            │+display_name│
    └─────────────┘             └─────────────┘
  + array: supplier, reseller, customer, sales-rep
  * virtual attribute (public method)

We disucssed/explained this in (part 2)[post_ruby_rails/rails_6_x_agnostic_associations_2/]

## Polymorphic

Coming soon

a model associated with several different models - serving a similar purpose in both cases

      ┌────────────┐          ┌───────────┐
      │            │╲       1 │           │
      │  Business  │ ○ ─ ─ ─ ┼│  Person   │
      │            │╱ 0..*    │           │
      └────────────┘          └───────────┘
            ╲│╱ *                * ╲│╱
             └───────────┬──────────┘
                         ┼ 1
                 ┌──────────────┐
                 │              │
                 │  Transaction │
                 │              │
                 └──────────────┘

A contact could be either a person or a business - but must be one or the other.

```bash
bin/rails g Contact roles:array business:references person:references
```

update the migration to ensure we have a role provided & relations:
```ruby
#
```

update the Contact model with the validations & flexible relations:
```ruby
# contact.rb
```

update the Person model and relations:
```ruby
# person.rb

```

update the Business model and relations:
```ruby
# business.rb

```


lets use seed a couple of people too - so it now looks like:
```ruby
# db/seed.rb
business = Business.create(legal_name: "Business")
company = Business.create(legal_name: "Company")

company.build_person(full_name: "Company Man")
company.save

person = Person.create(full_name: "Own Person")
```

Lets check the seed with:
```bash
bin/rails db:seed
```

Assuming this works, let's see the "/people" page:
```bash
bin/rails s
open localhost:3000/businesses/
```

Great - lets snapshot:
```bash
git add .
git commit -m "created person possibly related to the model"
```
