---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Lucky Framework with Crystal Language"
subtitle: "Basics of Lucky: Relationships, Types, Forms and Language Tweeks"
summary: "A simple but reasonably comprehensive overview of Lucky features - with the context of a 'mini-project'"
authors: ["btihen"]
tags: ["Relationships", "Basics", "Forms", "Components", "Routing", "Lucky", "Web Framework", "Crystal Language"]
categories: ["Code", "Lucky", "Crystal Language"]
date: 2021-05-02T01:01:53+02:00
lastmod: 2021-05-03T01:01:53+02:00
featured: false
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

## Why Lucky

Lucky offers all the features I use in Rails - but is type safe and faster than rails.  Lucky's focus is on stability (its not the fastest Crystal Framework, but it focuses on preventing run-time problems).  See: https://luckyframework.org/guides/getting-started/why-lucky for a full list of what Lucky aims to improve.


## Resources

This article is a collection of making sense of the following resources

- https://luckycasts.com/
- https://luckyframework.org/guides
- https://onchain.io/blog/lucky_tutorial
- https://github.com/andrewmcodes/awesome-lucky
- https://dev.to/hinchy/setting-up-a-crud-app-in-lucky-jo1

My goal is to have a simple tutorial for important basic features and orientation of the Lucky Framework - for myself and students I work with.


## installing Lucky

For more information see: https://luckyframework.org/guides/getting-started/installing

The `brew install` of lucky (on a MacOS) is bit broken, but the Linux install also works well on MacOS!

First be sure openssl and postgresql are installed and findable:

```
brew install openssl postgresql

# and depending on your shell either (if you don't know which it is safe to do both):
echo 'export PKG_CONFIG_PATH=/usr/local/opt/openssl/lib/pkgconfig' >>~/.zshrc
echo 'export PKG_CONFIG_PATH=/usr/local/opt/openssl/lib/pkgconfig' >>~/.bash_profile

# IMPORTANT - OPEN a new terminal

# or if you know what shell you are using you can reload it with source!
source ~/.bash_profile
```

Now install (or be sure ASDF is installed). https://asdf-vm.com/#/core-manage-asdf-vm

```
brew install asdf
# assuming bash
echo -e "\n. $(brew --prefix asdf)/asdf.sh" >> ~/.bash_profile
echo -e "\n. $(brew --prefix asdf)/etc/bash_completion.d/asdf.bash" >> ~/.bash_profile

# or zsh
echo -e "\n. $(brew --prefix asdf)/asdf.sh" >> ${ZDOTDIR:-~}/.zshrc
```

Now we add asdf plugin for crystal:
```
asdf plugin-add crystal https://github.com/asdf-community/asdf-crystal.git
```

For both Ruby and Crystal the following is also helpful:
```
echo "legacy_version_file = yes" >>~/.asdfrc
```

Lucky 0.27 needs Crystal 0.36.1 (not Crystal 1.0.0) - so we install it with:
```
asdf install crystal 0.36.1
```

I like to the local folder to crystal 0.36.1 (& the node version too) - this will allow use to install and run the lucky-cli tool
```
echo "cyrstal 0.36.1" >> .tool-versions
echo "node 14.16.0" >> .tool-versions
```

(but you can also just use: `asdf global crystal 0.36.1` - so you don't have to set the crystal version in every file you work in)!

Now let's install lucky_cli & also lucky
```
git clone https://github.com/luckyframework/lucky_cli
cd lucky_cli
git checkout v0.27.0
shards install

# if this following step fails (you probably forgot to reload your shell after the openssl lib path update)
crystal build src/lucky.cr

# make your compiled lucky_cli available everywhere
mv lucky /usr/local/bin
```

Now if you check your settings you should get:
```
lucky -v
# This should return 0.27.0

node -v
# should be 12.x or greater

yarn -v
# should be 1.x or greater

psql --version
# should be 10.x or greater
```


## Start a Lucky Project

Create your new Lucky project with the wizzard (just answer questions) - other options are at: https://luckyframework.org/guides/getting-started/starting-project

```
lucky init
cd {project_name}

# update the db settings in: `config/database.cr`

# if this step fails you may have forgotten to reload the shell after updating the openssl path
script/setup

# run lucky with:
lucky dev
```

Ok lets do an initial commit:
```
git add .
git commit -m "initial commit after create"
```

## Language Inflections

Let's make a little silly Human and Pets database/webpage:

The simplest way to generate is with: `lucky gen.resource.browser` its basically the same as `rails g scaffold`

So lets get started:
```
lucky gen.resource.browser Human name:String
```

OOPS - that generated the plural of `Human` as `Humen` instead of `Humans`!

lets clear all our incorrect files and fix this:
```
git clean -fd
```

Now let's create a new config file for inflections `config/inflect.cr` and enter:
```
# this probably isn't necessary for very long - but for now it is needed.
Wordsmith::Inflector.inflections.irregular("human", "humans")

# I like using persons (also a dictionary word) over people, to do this we need
# - first we have to remove the original setting by doing:
Wordsmith::Inflector.inflections.plurals.delete(/(p)erson$/i)
# - now we can override the original with our preference
Wordsmith::Inflector.inflections.irregular("person", "persons")

# if using `staff` as in human staff - then also add staff to uncountable:
Wordsmith::Inflector.inflections.uncountable(%w(staff))
```

Now if we try again we will have the same problem!  We need to remove our binaries and recompile lucky with our need config!  (I lost a lot of time on this detail)! Do this with:
```
rm -rf lib && rm -rf bin && shards update
```

Now we can try to create a new Resource again.


## Quick Lucky Test Tip

Lets quickly test our new config wiht `lucky exec` - type:
```
lucky exec
```

This gives you an edit in your cli and you can type a small amount of code and it will be compiles and print you the results - ie:
```
lucky exec
# then when vim or nano opens you can enter something like:

require "../../src/app.cr"

include Lucky::TextHelpers

pp pluralize(2, "human")
```

and hopefuly you get `2 humans` - cool - it works lets snapshot our changes.
```
git add .
git commit -m "language inflection updates and customization"
```


## Scaffold a Simple Resource

https://luckyframework.org/guides/command-line-tasks/built-in

Now if we try again (we are free to use human again):
```
lucky gen.resource.browser Human name:String
```

Now lets run the migration:
```
lucky db.migrate

# oops I haven't create the DB yet
lucky db.create

# now migrate
lucky db.migrate

# start lucky & test
lucky dev
```

Now log_in and create humans at the `/humans` url

Cool - lets snapshot:
```
git add .
git commit -m "First simple 'Human' resource with scaffold"
```


## Lucky HTML and RootPage Routing

https://luckycasts.com/videos/lucky-html-in-crystal
https://www.luckyframework.org/guides/http-and-routing/routing-and-params#root-page

If we look in `src/actions/home/index.cr` we see:
```
# src/actions/home/index.cr
class Home::Index < BrowserAction
  include Auth::AllowGuests

  get "/" do
    if current_user?
      redirect Me::Show
    else
      # html Landing::IndexPage
      html Lucky::WelcomePage
    end
  end
end
```

As we can see - when we are not logged in "/" points to `Lucky::WelcomePage` or whatever new landing page we make and when logged in we are pointed to the `Me::Show` page.

Let practice adding some `html` and add links to our expected resources:
- '/humans'
- '/pets'

So lets change this too and practice lucky html

We will add our list of resources - 'pets' and 'humans'.

So from looking at the existing html in `src/pages/me/show_page.cr` it's like a combo of haml and JS to create executable blocks with `{}` so I created the method: `private def resource_links` and tried out two methods of linking - not bad, but I figure it will take a bit of practice with this new format.  I don't know the reason behind this, since almost all web resources will need to be reformatted - but I assume it is pre-compiled and thus fast!


In the end I created this:
```
# src/pages/me/show_page.cr
class Me::ShowPage < MainLayout
  def content
    h1 "This is your profile:"
    h2 "Email:  #{@current_user.email}"
    resource_links
    helpful_tips
  end

  private def resource_links
    h2 "Available Resources"
    ul do
      li { a "Pet Owners", href: "/humans" }
      li { link_to_pets }
    end
  end

  private def helpful_tips
    h3 "Next, you may want to:"
    ul do
      li { link_to_authentication_guides }
      li "Modify this page: src/pages/me/show_page.cr"
      li "Change where you go after sign in: src/actions/home/index.cr"
    end
  end

  private def link_to_pets
    a "Pets", href: "/pets"
  end

  private def link_to_authentication_guides
    a "Check out the authentication guides",
      href: "https://luckyframework.org/guides/authentication"
  end
end
```

lets test it out:
```
lucky dev
```

cool - good enough for now.
```
git add .
git commit -m "added html links to user_home_page 'me'"
```


## Create a Related Model

https://www.luckyframework.org/guides/database/models#belongs-to
https://www.luckyframework.org/guides/database/models#has-many-one-to-many
https://luckyframework.org/guides/database/migrations#associations

Unfortunately, the Lucky generators don't understand `belongs_to` so we will need to do a few extra tweeks -- since we can't do something like human:belongs_to or human:references like with Rails.

So if we want to scaffold "pets" now and have them belong to humans (and humans can have many pets) - we first do:
```
lucky gen.resource.browser Pet name:String breed:String species:String age:Int32 house_trained:Bool
```

Now let's setup the relationships:

First we need to update the migration with the human foreign_key using: `add_belongs_to`

So we need to update our pets migration to:
```
# db/migrations/yyyymmddxxxxxx_create_pets.cr
class CreatePets::V20210502100912 < Avram::Migrator::Migration::V1
  def migrate
    create table_for(Pet) do
      primary_key id : Int64
      add_timestamps
      add name : String
      add breed : String
      add species : String
      add age : Int32
      add house_trained : Bool

      # When the associated human is deleted, their pets are also deleted
      # because we set on_delete: :cascade
      add_belongs_to human : Human, on_delete: :cascade    # relationship - newly added
    end
  end

  def rollback
    drop table_for(Pet)
  end
end
```

Now that the pets database table will is correct - lets update the pet model too.
This is straight-forward we just need to add `belongs_to human : Human` in the model file so it changes to:
```
# src/models/pet.cr
class Pet < BaseModel
  table do
    column name : String
    column breed : String
    column species : String
    column age : Int32
    column house_trained : Bool

    belongs_to human : Human     # relationship - newly added
  end
end
```

now we need to add `has_many` to the `Human` model.  So we change it to:
```
# src/models/human.cr
class Human < BaseModel
  table do
    column name : String

    has_many pets : Pet    # relationship - newly added
  end
end
```

Now we can migrate:
```
lucky db.migrate
```

## Seed Files

https://dev.to/hinchy/setting-up-a-crud-app-in-lucky-jo1
https://luckyframework.org/guides/database/database-setup#seeding-data

Now we can create seed files and be sure our basic relations work:
```

```

and run the seeds with:
```

```


## Simple Lucky Forms (in pages instead of shared)

https://luckycasts.com/videos/component-basics
https://luckycasts.com/videos/lucky-html-in-crystal
https://luckyframework.org/guides/frontend/html-forms
https://dev.to/hinchy/setting-up-a-crud-app-in-lucky-jo1
https://luckyframework.org/guides/frontend/html-forms#shared-components


Lets test the web page
```
lucky dev
```

and go to the url `/pets` and create a **Pet**.

We discover we have problems - validation errors.
- Type mismatches (crystal is strongly typed - but the form generator ignores this - so we need to adjust by hans)
- Required human_id is missing (the generator isn't aware of `belongs_to`)

I didn't find lots of Documentation or examples on Components, but I did find this article - which got me started on Lucky html and forms:
https://dev.to/hinchy/setting-up-a-crud-app-in-lucky-jo1

After I figured out how to update FormComponents I found this: https://luckycasts.com/videos/component-basics - I'll go back and view this!

This got me going!  However, one difficulty I had was the Boolean field `house_trained` - I tried both Checkboxes and Radio Buttons, but I kept getting `overload` errors (which I finally realized were type mis-matches - you can't send text into a Boolean field).  So I settled on a select_list where I can present a tuple with a "human readable value" and a "model value".

So in the end my first draft form looked like:
```
# src/pages/pets/new_page.cr
class Pets::NewPage < MainLayout
  needs operation : SavePet
  quick_def page_title, "New Pet"

  def content
    h1 "New Pet"
    render_pet_form(operation)
  end

  def render_pet_form(op)
    # comment out the form component for now
    # form_for Pets::Create do
    #   # Edit fields in src/components/pets/form_fields.cr
    #   mount Pets::FormFields, op

    #   submit "Save", data_disable_with: "Saving..."
    # end

    form_for Pets::Create do
      div do
        label_for op.name
        text_input op.name
      end
      div do
        label_for op.species
        select_input(op.species, class: "custom-select") do
          select_prompt("Select")
          options_for_select(op.species, [{"Cat", "cat"}, {"Dog", "dog"}])
        end
      end
      # eventually allow for this to be blank
      # eventually allow a dropdown list to be dependent on species
      div do
        label_for op.breed
        text_input op.breed
      end
      div do
        label_for op.age
        number_input(op.age, class: "custom-input", min: "0", max: "99")
      end
      div do
        label_for op.house_trained
        select_input(op.house_trained, class: "custom-select") do
          select_prompt("Select")
          options_for_select(op.house_trained, [{"yes", true}, {"No", false}])
        end
      end
      div do
        label_for op.human_id
        select_input op.human_id do
          options_for_select(op.human_id, humans_for_select)
        end
      end
      submit "Save Pet"
    end
  end

  # find all the humans and create a tuple of the name and id - displayed and passed to model
  private def humans_for_select
    HumanQuery.new.map do |human|
      { human.name, human.id }
    end
  end
end
```

of course this isn't shared by the `edit` page, but it is still helpful to see the Lucky way to create html.

So after a while I figured out how to revert this code and use SharedForms (I think this is a form of FrontEnd Components).

Lets test again:
```
lucky dev
```

Cool it works as I expect
```
git add .
git commit -m "a working 'new' html form - not shared"
```

## Shared Web Form - Component

https://luckycasts.com/videos/component-basics
https://luckyframework.org/guides/frontend/html-forms
https://luckyframework.org/guides/frontend/html-forms#shared-components

With a little more experience with Lucky HTML lets try the component forms again at `src/components/pets/form_fields.cr` (so lets revert: `src/pages/pets/new_page.cr` back to:
```
# src/pages/pets/new_page.cr
class Pets::NewPage < MainLayout
  needs operation : SavePet
  quick_def page_title, "New Pet"

  def content
    h1 "New Pet"
    render_pet_form(operation)
  end

  def render_pet_form(op)
    form_for Pets::Create do
      # Edit fields in src/components/pets/form_fields.cr
      mount Pets::FormFields, op

      submit "Save", data_disable_with: "Saving..."
    end
  end
end
```

Once I had build the first form and understood the errors - so the same form as a form_component looks like:
```
# src/components/pets/form_fields.cr
class Pets::FormFields < BaseComponent
  needs operation : SavePet

  def render
    mount Shared::Field, operation.name, &.text_input(autofocus: "true", attrs: [:required])
    mount Shared::Field, operation.species do |input_html|
      input_html.select_input append_class: "select-input" do
        select_prompt("Select")
        options_for_select operation.species, [{"Dog", "dog"}, {"Cat", "cat"}]
      end
    end
    mount Shared::Field, operation.breed  # default setting: &.text_input(attrs: [:required])
    mount Shared::Field, operation.age, &.number_input(append_class: "custom-input", min: "0", max: "99")
    mount Shared::Field, operation.house_trained do |input_html|
      input_html.select_input append_class: "select-input" do
        select_prompt("Select")
        options_for_select operation.house_trained, [{"yes", true}, {"no", false}]
      end
    end
    mount Shared::Field, operation.human_id do |input_html|
      input_html.select_input append_class: "select-input" do
        select_prompt("Select")
        options_for_select operation.human_id, options_for_humans
      end
    end
  end

  private def options_for_humans
    HumanQuery.new.map do |human|
      { human.name, human.id }
    end
  end
end
```

Lets test again:
```
lucky dev
```

cool - lets snapshot:
```
git add .
git commit -m "working shared form component with a variety of types"
```

## Redirect after Create / Update to Index

https://luckyframework.org/guides/http-and-routing/routing-and-params

I find it annoying after creating and updating a resource to have to then manually go back to the index page from the show page.

In lucky the routing/controll happens in the `action` files.

To change what happens after creating and updating a Pet we simply change `src/actions/pets/create.cr` to:
```
# src/actions/pets/create.cr
class Pets::Create < BrowserAction
  post "/pets" do
    SavePet.create(params) do |operation, pet|
      if pet
        flash.success = "The record has been saved"
        html IndexPage, pets: PetQuery.new  # new action (copied from index)
        # redirect Show.with(pet.id)        # old no longer wanted
      else
        flash.failure = "It looks like the form is not valid"
        html NewPage, operation: operation
      end
    end
  end
end
```

And update `src/actions/pets/update.cr` is similarly easy:
```
# src/actions/pets/update.cr
class Pets::Update < BrowserAction
  put "/pets/:pet_id" do
    pet = PetQuery.find(pet_id)
    SavePet.update(pet, params) do |operation, updated_pet|
      if operation.saved?
        flash.success = "The record has been updated"
        html IndexPage, pets: PetQuery.new
        # redirect Show.with(updated_pet.id)
      else
        flash.failure = "It looks like the form is not valid"
        html EditPage, operation: operation, pet: updated_pet
      end
    end
  end
end
```

I appreciate how explicit these are!

## Display Validation Errors

If we leave some fields out - Lucky gives us validation errors - all fields appear to be required without explicitly allowing nils - but we don't see them with our default form.  Lets fix that.


## Add Validations

Let's add a few custom validations:
- minimal pet_name length
- numeric range


## Tests for our Validations

Now that we have some logic lets add some tests


## Optional Fields

Often a **breed** is unknown - we could just add an `unknown` value, but that's silly, lets figure out how to work with unknown / unneeded data and allow nil in our `breed` field.


## Bulma Integration

Integrate CSS Frameworks

## View Components

https://luckycasts.com/videos/component-basics
https://luckyframework.org/guides/frontend/rendering-html#creating-and-using-components


## Tailwind Integration

https://luckycasts.com/videos/tailwind-css

Let's make the pages a bit nicer


## HTML to Lucky formatter

https://luckyframework.org/html

If we want to create some more complex pages with tailwind - lets use the converted to help.


## Dynamic Front-end - Selections Dependencies (AlpineJS / StimulusJS) ?

https://luckycasts.com/videos/stimulus-js

Make the breed list, dependent on the species list
Lets change the Front-End language on the fly
Lets make the new TailwindUI menu bar have the dynamic features.



## Has many through

https://www.luckyframework.org/guides/database/models#has-many-through-many-to-many



## Polymorphic Relationships

https://www.luckyframework.org/guides/database/models#polymorphic-associations

One reason I favor Lucky is the database `Avram` supports polymorphic relationships - which seems to come up a lot in my code - so lets see how to get it working and support multiple types:



## Optional Relations



## Internationalization (i18n)

https://luckycasts.com/videos/translations
https://luckyframework.org/guides/frontend/internationalization


## Dynamic i18n in Front-End?


## Components (with scopes)


## Lucky Code Scopes


## Resource Authorization

https://github.com/stephendolan/pundit


## Web Sockets

For now something like **LiveView** and **Hotwire** are not yet integrated into lucky - its build your own.

https://github.com/cable-cr/cable
https://github.com/luckyframework/lucky/issues/554


## Deploying Lucky & ENV

https://fullstackstanley.com/read/categories/lucky-framework/


## Security (Alternatives)

https://github.com/grottopress/shield
