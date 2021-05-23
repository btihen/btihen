---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Rails 6.x - Framework Agnostic Associations - part 1"
subtitle: "Framework Agnostic Polymorphism - allowing Ruby Code t"
summary: "Framework Agnostic Polymorphism - allowing Rails Code to easily interoperate smoothly with other languages"
authors: ["btihen"]
tags: ['Rails', 'Hotwire', 'SPA', 'WebSocket', 'realtime', 'flash message']
categories: []
date: 2021-05-19T01:57:00+02:00
lastmod: 2021-05-19T01:57:00+02:00
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

## Overview

In this case, I want to model a contact list of businesses and people.  Some people will be associated with a company.  Additionally, we will track transactions with each person and business.

The basic model will then look something like:

                  ┌───────────────┐
                  │    Contact    │
                  │ (roles:array) │
                  │(display_name)*│  *virtual attribute
                  └───────────────┘
                          ┼ 1
             ┌────────────┴─────────────┐
            ╱│╲ *                    * ╱│╲
    ┌───────────────┐          ┌───────────────┐
    │    Business   │╲       1 │    Person     │
    │  (legal_name) │ ○ ─ ─ ─ ┼│  (full_name)  │
    │(display_name)*│╱ 0..*    │(display_name)*│
    └───────────────┘          └───────────────┘
            ╲│╱ *                    * ╲│╱
             └────────────┬─────────────┘
                          ┼ 1
                  ┌───────────────┐
                  │  Transaction  │
                  │    (notes)    │
                  │               │
                  └───────────────┘
                Created with Monodraw

## Create a default Rails app

```bash
rails new rails_poly
cd rails_poly
bin/rails db:create
bin/rails db:migrate
git add .
git commit -m "initial commit"
```

## Starting Simple - optional relations

### Build Businesses

Lets start with the simple relationship between businesses and people:

      ┌────────────┐          ┌───────────┐
      │            │╲       1 │           │
      │  Business  │ ○ ─ ─ ─ ┼│  Person   │
      │(legal_name)│╱ 0..*    │(full_name)│
      └────────────┘          └───────────┘

For expedience, I'll use scaffolds:

Generating a simple business model.
```bash
rails g scaffold Business legal_name
```

Lets adjust the migration to require the business' legal name, by adding `null: false` to the name:
```ruby
# db/migrate/20210516080420_create_businesses.rb
class CreateBusinesses < ActiveRecord::Migration[6.1]
  def change
    create_table :businesses do |t|
      t.string :legal_name, null: false

      t.timestamps
    end
  end
end
```


Now we will validate the business' name in the model:
```ruby
# app/models/business.rb
class Business < ApplicationRecord
  validates :legal_name, presence: true
end

```
Now lets be sure we can migrate:
```bash
bin/rails db:migrate
```

lets use seed to quickly check our models and relations (& get an idea of how to use them):
```ruby
# db/seeds.rb
business = Business.create(legal_name: "Business")
```

Lets check the seed with:
```bash
bin/rails db:seed
```

Assuming this works, let's see the "/businesses" page:
```bash
bin/rails s
open localhost:3000/businesses/
```

Great - lets snapshot:
```bash
git add .
git commit -m "created business model"
```

### Build People

Now let's build the person model and its relations to businesses.
```bash
rails g scaffold Person full_name business:references
```

In this case we want the person to optionally be a member of a business, so lets update the both the models and the migration.  Starting with the migration, we need to remove `null: false` in the foreign key, and add that to the name - so it should now look like:
```ruby
# db/migrate/20210516080414_create_people.rb
class CreatePeople < ActiveRecord::Migration[6.1]
  def change
    create_table :people do |t|
      t.string :full_name, null: false
      t.references :company, foreign_key: true

      t.timestamps
    end
  end
end
```

Now lets adjust the person model - we'll make the relation optional with `optional: true` and require the name with the validation `validates :full_name, presence: true`, so it should now look like:
```ruby
# app/models/person.rb
class Person < ApplicationRecord
  belongs_to :company, optional: true

  validates :full_name, presence: true
end
```

And lets let the Business know it can have lots of people with `has_many :people` - now the model will look like:
```ruby
# app/models/business.rb
class Business < ApplicationRecord
  has_many :people

  validates :legal_name, presence: true
end
```

Lets check the migrations work:
```bash
bin/rails db:migrate
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

Now lets check our pages again:
```bash
bin/rails s
open localhost:3000
```

Lets check the index pages

On the business page it would be nice to see how many employees - so we can update the model with:
```ruby
# app/models/business.rb
class Business < ApplicationRecord
  has_many :people

  validates :legal_name, presence: true

  def people_count
    people.count
  end
end
```

And now `people_count` is added as a virtual attribute (as well as all other business fields because of `'businesses.*`) - now we can use in our view using = `<td><%= business.people_count %></td>` so now it would look something like:
```ruby
# app/views/businesses/index.html.erb
<h1>Businesses</h1>

<table>
  <thead>
    <tr>
      <th>Legal name</th>
      <th>Employee Count</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @businesses.each do |business| %>
    <tr>
      <td><%= business.legal_name %></td>
      <td><%= business.people_count %></td>
      <td><%= link_to 'Show', business %></td>
      <td><%= link_to 'Edit', edit_business_path(business) %></td>
      <td><%= link_to 'Destroy', business, method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
    <% end %>
  </tbody>
</table>
```

and on the '/people' page it would be nice to see there business name instead of id.

so in the model:
```ruby
# app/model/person.rb
class Person < ApplicationRecord
  belongs_to :business, optional: true

  validates :full_name, presence: true

  def associated_business_name
    business&.legal_name
  end
end
```

and in the index view:
```ruby
# app/views/people/index.html.erb
<table>
  <thead>
    <tr>
      <th>Full name</th>
      <th>Business</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @people.each do |person| %>
    <tr>
      <td><%= person.full_name %></td>
      <td><%= person.associated_business_name %></td>
      <td><%= link_to 'Show', person %></td>
      <td><%= link_to 'Edit', edit_person_path(person) %></td>
      <td><%= link_to 'Destroy', person, method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
    <% end %>
  </tbody>
</table>
```

to show all employees on the business show page we can do:
```ruby
# app/views/businesses/show.html.erb
<p>
  <strong>Legal name:</strong>
  <%= @business.legal_name %>
</p>

<table>
  <thead>
    <tr>
      <th>Employee</th>
    </tr>
  </thead>
  <tbody>
    <% @business.people.each do |person| %>
    <tr><td>person.full_name</td></tr>
    <% end %>
  </tbody>

</table>
```

And now lets look for n+1 queries - to do that we will create many records in the seeds file:
```ruby
# db/seeds.rb
business = Business.create(legal_name: "Business")
company  = Business.create(legal_name: "Company")
boss_man = Person.create(full_name: "Company Man", business: company)
person = Person.create(full_name: "Own Person")

# larger numbers (look for n+1 lookups)
50.times do |business_number|
  company  = Business.create(legal_name: "Company #{business_number}")
  business_number.times do |employee_number|
    Person.create(full_name: "Employee #{employee_number}",
                  business: company)
  end
end
```

Now when we visit '/people' we see an n+1 (to look up the business to get the business name) - this is an easy fix with a pre-load in the controller - just add `.include(:business)` to the query - now the index method will look like
```ruby
# app/controllers/people_controller.rb
class PeopleController < ApplicationController

  def index
    @people = Person.include(:business).all
  end
```



Fix n+1 lookups - for the business employee count is a bit trickier - to avoid lots of look ups we need the db to do the count and add the count as a virtual attribute - this is done with the following query:
```ruby
# app/controllers/people_controller.rb
class BusinessController < ApplicationController

  def index
    # businesses = Business.all  # (N+1 when using referring to people)
    # select must go last or it gets lost / overwritten
    @businesses = Business.joins(:people)
                          .group('businesses.id')
                          .select('businesses.*, count(people.id) as people_count')
  end
```

to avoid confusion - lets rename the method in the class to `employee_count`:
```ruby
# app/models/business.rb
class Business < ApplicationRecord
  has_many :people

  validates :legal_name, presence: true

  def employee_count
    people.count
  end
end
```


lets run the seeds again:
```bash
bin/rails db:seed
```

cool now when we look at the log we just have one query instead of many!

Now let's make the people form to associate a business by name instead of the id!
```ruby
# app/views/people/_form.html.erb
<%= form_with(model: person) do |form| %>
  <% if person.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(person.errors.count, "error") %> prohibited this person from being saved:</h2>

      <ul>
        <% person.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= form.label :full_name %>
    <%= form.text_field :full_name %>
  </div>

  <div class="field">
    <%= form.label :business %>
    <%= form.select :business_id,
                    Business.all.collect { |b| [ b.legal_name, b.id ] },
                    prompt: "Select One", include_blank: true %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

Great - lets snapshot:
```bash
git add .
git commit -m "created person related to businesses - w/o n+1"
```

## Polymorphic (STI) - sometime called inverse polymorphic

              ┌───────────────┐
              │    Contact    │
              │  (functions)+ │ + supplier, reseller, customer, sales-rep
              │(display_name)*│ * virtual attribute
              └───────────────┘
                      ┼ 1
         ┌────────────┴─────────────┐
        ╱│╲ *                    * ╱│╲
┌───────────────┐          ┌───────────────┐
│    Business   │╲       1 │    Person     │
│  (legal_name) │ ○ ─ ─ ─ ┼│  (full_name)  │
│(display_name)*│╱ 0..*    │(display_name)*│
└───────────────┘          └───────────────┘

A contact could be either a person or a business - but must be one or the other.

### Migration and Relationships

Rails doesn't have a built-in array migration, so we use string and then we change the migration:
```bash
bin/rails g scaffold Contact functions:string business:references person:references
```

Now update the migration to ensure we have a functions as an array & relations as Foreign keys (but optional). Since there we only want/need one of the two foreign_keys at a time they must be nullable and we need to change roles to an array - so now:
```ruby
# db/migrate/20210519205042_create_contacts.rb
class CreateContacts < ActiveRecord::Migration[6.1]
  def change
    create_table :contacts do |t|
      t.string :functions, array: true, null: false, default: []
      t.references :business, foreign_key: true
      t.references :person, foreign_key: true

      t.timestamps
    end
  end
end
```

update the Contact model with the validations & flexible relations - we also want to be able to refer to the sub-model by one name we will call that `contactable` - so now the model will look like:
```ruby
# app/models/contact.rb
class Contact < ApplicationRecord
  belongs_to :business, optional: true
  belongs_to :person, optional: true

  VALID_FUNCTIONS_LIST = %w(supplier reseller customer sales-rep)

  validate :validate_relationship_functions
  validate :validate_belongs_to_one_and_only_one_foreign_key

  def contactable
    business || person
  end

  private

  # be sure we have the variable, it is an Array & all elements are in the valid list
  def validate_relationship_functions
    return if functions.present? && functions.is_a?(Array)
              functions.all? { |role| VALID_FUNCTIONS_LIST.include?(role.to_s) }

    errors.add :functions, "must be ONE or MORE of the following options: #{VALID_FUNCTIONS_LIST.join(',')}"
  end

  # exclusive or (XOR) is true if one or the other is true, but not when both are true
  # we could get a model (or possibly an id)
  def validate_belongs_to_one_and_only_one_foreign_key
    return if business.present? ^ person.present? ^ business_id.present? ^ person_id.present?

    # add to base since, the error could be either field.
    errors.add :base, 'must belong to ONE business or person, but not both'
  end

end
```

update the Person model and relations and enforce every person is a member of the contact list - with a contact role:
```ruby
# app/models/person.rb
class Person < ApplicationRecord
  has_one :contact
  belongs_to :business, optional: true

  validates :contact, presence: true
  validates :full_name, presence: true
end
```

update the business model and relations and enforce every business is a member of the contact list - with a contact role:
```ruby
# # app/models/business.rb
class Business < ApplicationRecord
  has_one :contact
  has_many :people

  validates :contact, presence: true
  validates :legal_name, presence: true
end
```


Lets check the seed with:
```bash
bin/rails db:migrate
```

If we go to a person or business we can no longer make changes - they need to have an associated Contact.
We'll start by rolling back the last migration and fixing it with (we can use the logic in the seeds to guide us in the Business/Person creation controller):

```bash
bin/rails db:rollback
```

we need to fix the old relations in the migration (or simply drop the database and reseed it) - but given this is to article is find cross-framework -- 'real-world' techniques - let's be sure the existing records stay useful.  We will assume a business is a supplier, a person associated with a business is a sales-rep, and unassociated people are customers.
```ruby
# db/migrate/20210519205042_create_contacts.rb
class CreateContacts < ActiveRecord::Migration[6.1]
  def change
    create_table :contacts do |t|
      t.string :functions, array: true, null: false, default: []
      t.references :business, foreign_key: true
      t.references :person, foreign_key: true

      t.timestamps
    end

    # add a contact for each existing company
    businesses = Business.joins(:people)
                         .group('businesses.id')
                         .select('businesses.*, count(people.id) as people_count')
    businesses.each do |business|
      functions = if business.people_count < 10
                    ['supplier']
                  elsif business.people_count < 20
                    ['reseller']
                  elsif business.people_count < 30
                    ['supplier', 'reseller']
                  end
      Contact.create!(functions: functions, business: business)
    end

    # add a contact for each existing person
    Person.all.each do |person|
      functions = if person.business
                    ['sales_rep']
                  else
                    ['customer']
                  end
      Contact.create!(functions: functions, person: person)
    end
  end
end
```

Lets the existing models now:
```bash
bin/rails db:migrate
```

OK - we are in business lets update our seed file too:
```ruby
# db/seed.rb
# create small business w/o employees
20.times do |num|
  business = Business.create(legal_name: "Business #{num}",
                             contact: Contact.new(functions: ['supplier']))
end

# create individuals
20.times do |num|
  person = Person.create(full_name: "Individual #{num}",
                            contact: Contact.new(functions: ['customer']))
end

# create big companies with employees
20.times do |bus_num|
  functions = if bus_num < 3
                ['supplier']
              elsif bus_num< 5
                ['reseller']
              elsif bus_num < 8
                ['supplier', 'reseller']
              else
                %w[supplier reseller customer]
              end
  company  = Business.create(legal_name: "Company #{bus_num}",
                             contact: Contact.new(functions: functions))

  bus_num.times do |emp_num|
    Person.create(full_name: "Employee #{bus_num}-#{emp_num}",
                  business: company,
                  contact: Contact.new(functions: ['sales-rep']))
  end
end
```

Lets check the seed with:
```bash
bin/rails db:seed
```

Great all works!

### Lets make the index page more useful

When we visit the contacts page we would like more than the ids - but we need a unified way to present that info so let's add a display_name so we can show the name of the primary model, if a person we would like to know the associated business if present and if a company we would like the employee_count so we will delegate these to the sub-models.

Let's update contact first by adding:
```ruby
  # this references our existing contactable
  delegate :display_name, :associated_business_name, :employee_count,
           to: :contactable

  def contactable
    business || person
  end
```

So now the contact model will look like (with validations)
```ruby
# app/models/contact.rb
class Contact < ApplicationRecord
  belongs_to :business, optional: true
  belongs_to :person, optional: true

  VALID_FUNCTIONS_LIST = %w(supplier reseller customer sales-rep)

  validate :validate_relationship_functions
  validate :validate_belongs_to_one_and_only_one_foreign_key

  delegate :display_name, :associated_business_name, :employee_count,
           to: :contactable

  def contactable
    business || person
    # would memoize be valuable here?
    # @contactable ||= (business || person)
  end

  private

  # be sure we have the variable, it is an Array & all elements are in the valid list
  def validate_relationship_functions
    return if functions.present? && functions.is_a?(Array)
              functions.all? { |role| VALID_FUNCTIONS_LIST.include?(role.to_s) }

    errors.add :functions, "must be ONE or MORE of the following options: #{VALID_FUNCTIONS_LIST.join(',')}"
  end

  # exclusive or (XOR) is true if one or the other is true, but not when both are true
  # we could get a model (or possibly an id)
  def validate_belongs_to_one_and_only_one_foreign_key
    return if business.present? ^ person.present? ^ business_id.present? ^ person_id.present?

    # add to base since, some forms may not have the person/business fields
    errors.add :base, 'must belong to ONE business or person, but not both'
    # errors.add :contactable, 'must belong to a business or a person'
  end
end
```

Lets update the models to provide the needed info

Business now will look like:
```ruby
# app/models/business.rb
class Business < ApplicationRecord
  has_one :contact
  has_many :people

  validates :contact, presence: true
  validates :legal_name, presence: true

  def display_name
    legal_name
  end

  def employee_count
    people.count
  end

  def associated_business_name
    ""
  end
end
```

And person will look like:
```ruby
# app/models/person.rb
class Person < ApplicationRecord
  has_one :contact
  belongs_to :business, optional: true

  validates :contact, presence: true
  validates :full_name, presence: true

  def display_name
    full_name
  end

  def employee_count
    nil  # person count has no meaning under person
  end

  def associated_business_name
    business&.display_name
  end
end
```

Now lets update the index view to show our new info:
```ruby
<h1>Contacts</h1>

<table>
  <thead>
    <tr>
      <th>Person/Business</th>
      <th>Employee Count</th>
      <th>Contact Name</th>
      <th>Business Name</th>
      <th>Relationships</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @contacts.each do |contact| %>
    <tr>
      <td><%= contact.contactable.class.name %></td>
      <td><%= contact.employee_count %></td>
      <td><%= contact.display_name %></td>
      <td><%= contact.associated_business_name %></td>
      <td><%= contact.functions %></td>
      <td><%= link_to 'Show', contact %></td>
      <td><%= link_to 'Edit', edit_contact_path(contact) %></td>
      <td><%= link_to 'Destroy', contact, method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
    <% end %>
  </tbody>
</table>
```

Now we see another n+1 query - we will fix the main part - but not the employee count this time:
```ruby
class ContactsController < ApplicationController
  def index
    # @contacts = Contact.all
    @contacts = Contact.includes(:business).includes(:person).all
  end
```

Cool now the page is usable (a bit long but we will ignore that)

### Lets be sure we can create new contacts

I usually use an input model (for more flexibility), but for now I will use nested_params.
A few articles on nested params and nested fields:

* https://www.youtube.com/watch?v=PYYwjTlcoa4
* https://www.pluralsight.com/guides/ruby-on-rails-nested-attributes
* https://levelup.gitconnected.com/rails-nested-forms-in-three-steps-5580f0ad0e
* https://levelup.gitconnected.com/handling-nested-attributes-with-a-has-many-through-association-with-rails-api-f91729547ea5

To start we will tell the contacts model that it can create nested models with do by adding:
```ruby
  accepts_nested_attributes_for :business
  accepts_nested_attributes_for :person
```

so now now the contact model looks like:
```ruby
# app/models/contact.rb
class Contact < ApplicationRecord
  belongs_to :business, optional: true
  belongs_to :person, optional: true

  accepts_nested_attributes_for :business
  accepts_nested_attributes_for :person

  VALID_FUNCTIONS_LIST = %w(supplier reseller customer sales-rep)

  validate :validate_relationship_functions
  validate :validate_belongs_to_one_and_only_one_foreign_key

  delegate :display_name, :associated_business_name, :employee_count,
           to: :contactable

  def contactable
    business || person
    # would memoize be valuable here?
    # @contactable ||= (business || person)
  end

  private

  # be sure we have the variable, it is an Array & all elements are in the valid list
  def validate_relationship_functions
    return if functions.present? && functions.is_a?(Array)
              functions.all? { |role| VALID_FUNCTIONS_LIST.include?(role.to_s) }

    errors.add :functions, "must be ONE or MORE of the following options: #{VALID_FUNCTIONS_LIST.join(',')}"
  end

  # exclusive or (XOR) is true if one or the other is true, but not when both are true
  # we could get a model (or possibly an id)
  def validate_belongs_to_one_and_only_one_foreign_key
    return if business.present? ^ person.present? ^ business_id.present? ^ person_id.present?

    # add to base since, some forms may not have the person/business fields
    errors.add :base, 'must belong to ONE business or person, but not both'
    # errors.add :contactable, 'must belong to a business or a person'
  end
end
```

In the controller we need to create models as part of @contact to allow nested-fields - which feed the nested attributes. to allow the new information in via strong params:
```ruby
# app/controllers/contacts_controller.rb
  def new
    @contact = Contact.new
    # add empty sub-models for our form
    @contact.person = Person.new
    @contact.business = Business.new
  end

  # update strong params to accept the sub-model attributes
  # sub-models from nested-forms feeding nested_atttributes in the model
  # take the form <model_name>_attributes
  # `functions` is an empty array since it is taking a list of values
  # person_attributes & business_attributes - need to include the list of attributes to accept!
  # so in our case:
  def contact_params
    contact_attribs = params.require(:contact)
                            .permit(functions: [],  # is empty - takes a list of values
                                    person_attributes: [:full_name],  # needs to include the list of attributes to accept
                                    business_attributes: [:legal_name])
  end
```

update the contact form to tie this all together by adding our nested forms:
```ruby
  <div class="field-group">
    <h2>Create your Contact: a Person or a Business</h2>

    <h3>Business</h3>
    <%= form.fields_for :business, Business.new do |f| %>
      <%= f.label :legal_name %>
      <%= f.text_field :legal_name %>
    <% end %>

    <h3>Person</h3>
    <%= form.fields_for :person, Person.new do |f| %>
      <%= f.label :full_name %>
      <%= f.text_field :full_name %>
    <% end %>
  </div>
```

We will also need to make the list of possible relationship functions a multi-select - I always forget the format -- so remember BOTH {} are required when using multi-select!!  The first one is for normal drop-down select options -- like include_blank, the second one is where the multi-select must go!

This looks like:
```ruby
  <div class="field">
    <%= form.label :functions %>
    <%= form.select :functions,
                    options_for_select(Contact::VALID_FUNCTIONS_LIST,
                                      selected: Contact::VALID_FUNCTIONS_LIST.second),
                                      {}, #{:include_blank => 'None'},
                                      {:multiple => true, size: 3} %>
  </div>
```

so now the template looks like:
```ruby
# app/views/contacts/_form.html.erb
<%= form_with(model: contact) do |form| %>
  <% if contact.errors.any? %>
  <div id="error_explanation">
    <h2><%= pluralize(contact.errors.count, "error") %> prohibited this contact from being saved:</h2>

    <ul>
      <% contact.errors.each do |error| %>
      <li><%= error.full_message %></li>
      <% end %>
    </ul>
  </div>
  <% end %>

  <div class="field">
    <%= form.label :functions %>
    <%= form.select :functions,
                    options_for_select(Contact::VALID_FUNCTIONS_LIST,
                                      selected: Contact::VALID_FUNCTIONS_LIST.second),
                                      {}, #{:include_blank => 'None'},
                                      {:multiple => true, size: 3} %>
  </div>

  <div class="field-group">
    <h2>Create your Contact: a Person or a Business</h2>

    <h3>Business</h3>
    <%= form.fields_for :business, Business.new do |f| %>
      <%= f.label :legal_name %>
      <%= f.text_field :legal_name %>
    <% end %>

    <h3>Person</h3>
    <%= form.fields_for :person, Person.new do |f| %>
      <%= f.label :full_name %>
      <%= f.text_field :full_name %>
    <% end %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

Now when we try `/contacts` we notice one more problem - it is always invalid - rails automatically add a leading "" in an array input list :( - so we will have to clean this up in the strong params.  In this case we are working with param objects not a hash so we will do an in-place update (removal of "") using:
```ruby
  contact_attribs["functions"].reject! {|f| f.blank? }
  contact_attribs
```

 we also need to be sure in our case we only send the params of the business or the person, but not both - since we are only creating one.  So we will remove whichever one is empty - also with an in-place update - using:
```ruby
    # find and set to nil the model without params
    if contact_attribs["person_attributes"]
      # since we only have one param we can do
      contact_attribs["person_attributes"] = nil if contact_attribs["person_attributes"]["full_name"].blank?
    end

    if contact_attribs["business_attributes"]
      # assuming we had multiple params the test is easier with:
      contact_attribs["business_attributes"] = nil if contact_attribs["business_attributes"].to_h.all? {|key,value| value.blank?}
    end

    # remove the nested attributes set to nil so contact will only create the desired associated model
    contact_attribs.reject! {|key, value| value.blank? }
    contact_attribs
```

So now the full controller looks like:
```ruby
class ContactsController < ApplicationController
  before_action :set_contact, only: %i[ show edit update destroy ]

  def index
    # @contacts = Contact.all
    @contacts = Contact.includes(:business).includes(:person).all
  end

  def show
  end

  def new
    @contact = Contact.new
    @contact.person = Person.new
    @contact.business = Business.new
  end

  def edit
  end

  def create
    @contact = Contact.new(contact_params)

    respond_to do |format|
      if @contact.save
        format.html { redirect_to @contact, notice: "Contact was successfully created." }
        format.json { render :show, status: :created, location: @contact }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @contact.errors, status: :unprocessable_entity }
      end
    end
  end

  def update
    respond_to do |format|
      if @contact.update(contact_params)
        format.html { redirect_to @contact, notice: "Contact was successfully updated." }
        format.json { render :show, status: :ok, location: @contact }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @contact.errors, status: :unprocessable_entity }
      end
    end
  end

  def destroy
    @contact.destroy
    respond_to do |format|
      format.html { redirect_to contacts_url, notice: "Contact was successfully destroyed." }
      format.json { head :no_content }
    end
  end

  private

  # Use callbacks to share common setup or constraints between actions.
  def set_contact
    @contact = Contact.find(params[:id])
  end

  # Only allow a list of trusted parameters through.
  def contact_params
    # update strong params to accept the sub-model attributes
    # sub-models from nested-forms feeding nested_atttributes in the model
    # take the form <model_name>_attributes
    # `functions` is an empty array since it is taking a list of values
    # person_attributes & business_attributes - need to include the list of attributes to accept!
    # so in our case:
    contact_attribs = params.require(:contact)
                            .permit(functions: [],
                                    person_attributes: [:full_name],
                                    business_attributes: [:legal_name])
    # cleanup array - always delivers with [''] - :(
    # https://stackoverflow.com/questions/51341912/empty-array-value-being-input-with-simple-form-entries

    # easiest way in in-place replacement (given that params is now objects and not a hash), but that always makes me a bit nervous
    # https://stackoverflow.com/questions/20164354/rails-strong-parameters-with-empty-arrays
    # reject and replace in place
    contact_attribs["functions"].reject! {|f| f.blank? }

    # remove empty model attributes
    # contact_attribs["person_attributes"].reject {|key,value| value.blank?}
    if contact_attribs["person_attributes"]
      contact_attribs["person_attributes"] = nil if contact_attribs["person_attributes"]["full_name"].blank?
    end

    if contact_attribs["business_attributes"]
      contact_attribs["business_attributes"] = nil if contact_attribs["business_attributes"].to_h.all? {|key,value| value.blank?}
    end

    # have to remove nil attributes for models so nested attributes works correctly
    contact_attribs.reject! {|key, value| value.blank? }

    # return the attributes with the tidied array
    contact_attribs
  end
end
```

now when we try again:
```bash
bin/rails s
open localhost:3000/contacts/new
```

Cool - it works.  We could now do the same for the `/business/new` and `/people/new`, but we won't do that here in the article. Lets snapshot:
```bash
git add .
git commit -m "created person possibly related to the model"
```

## Polymorphic

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
