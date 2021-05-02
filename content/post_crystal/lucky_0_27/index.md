---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Lucky Framework with Crystal Language"
subtitle: "Basics of Lucky with Relationships, Types, Forms and Language Tweeks"
summary: ""
authors: ["btihen"]
tags: ["Relationships", "Basics"]
categories: ["Crystal", "Lucky", "Languages"]
categories: ["Code"]
date: 2021-04-25T01:01:53+02:00
lastmod: 2021-04-27T01:01:53+02:00
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


## Quick Test Tip

If you need to quickly check a small piece of code and you don't want to test the whole app you can type:
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
and hopefuly you get `2 humans`


## Create Lucky Code

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
mkdir lib bin
```

Now if we try again (we are free to use human again):
```
lucky gen.resource.browser Human name:String
```

## Create a Related Model

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

Now we can migrate and test.
```
lucky db.migrate
```

Now we can create seed files and be sure the basic relations work:
```

```


## Test with Web forms

Now when we log in and create an human at the `/humans` url

Now when we go to the `/pets` url we discover we have problems - validation errors.
- Type mismatches (crystal is strongly typed - but the form generator ignores this - so we need to adjust by hans)
- Required human_id is missing (the generator isn't aware of `belongs_to`)

So lets add the human_id to the pet form:
```
# src/components/pets/form_fields.cr
class Pets::FormFields < BaseComponent
  needs operation : SavePet

  def render
    mount Shared::Field, operation.name, &.text_input(autofocus: "true", attrs: [:required])
    mount Shared::Field, operation.breed
    mount Shared::Field, operation.species
    mount Shared::Field, operation.age
    mount Shared::Field, operation.house_trained
    mount Shared::Field, operation.human_id, &.number_input() # newly added
  end
end
```

We know we will have some type errors, but lets see if this fixes the required relationship error.

NOPE - now it says we need to allow this to be saved - so we need to add the `human_id` variable to: `src/operations/save_pet.cr` -- now it should look like:
```
# src/operations/save_pet.cr
class SavePet < Pet::SaveOperation
  permit_columns name, breed, species, age, house_trained, human_id
end
```

Let's test again and be sure to use the human_id of the existing human.

Cool, but this is inconvient, let's make the human a drop-down list that inserts the expected integer.  We can do this some extra form-code:
```
# src/components/pets/form_fields.cr
class Pets::FormFields < BaseComponent
  needs operation : SavePet

  def render
    mount Shared::Field, operation.name, &.text_input(autofocus: "true")
    mount Shared::Field, operation.breed
    mount Shared::Field, operation.species
    mount Shared::Field, operation.age
    mount Shared::Field, operation.house_trained

    # human name for the user to see and the human's db_id for Lucky
    mount Shared::Field, operation.human_id do |input_html|
      input_html.select_input append_class: "select-input" do
        options_for_select operation.human_id, options_for_humans
      end
    end
  end

  # create an array of tuples with the human_name and human id - ie: [{"bill", 1}, {"jane", 2}]
  private def options_for_humans
    OwnerQuery.new.map do |human|
      { human.name, human.id }
    end
  end
end
```

The private method `options_for_owners` queries the database and creates an array of tuples with the human's name to display and the human's db_id - ie:  [{"Bill", 1}, {"Jane", 2}] which is then used by the `do` which builds the `select` form.

If we test this we should now not only be free of the validation error on the relationships, but also have a way to ensure the user has appropriate values to enter.

Let's fix the house_training input - I tried variations on checkboxes and radio-buttons:
```
    # mount Shared::Field, operation.house_trained, &.checkbox("no", "yes", append_class: "custom-check")
    # mount Shared::Field, operation.house_trained, &.checkbox(0, 1, append_class: "custom-check")
    # mount Shared::Field, operation.house_trained, &.checkbox(false, true, append_class: "custom-check")
    # mount Shared::Field, operation.house_trained, &.radio(op.question_five, "Yes")
    # mount Shared::Field, operation.house_trained, &.radio(op.question_five, "No")
```
but it seems these will only pass strings - got no end of errors looking like:
`Error: no overload matches 'Shared::Field(Bool)#checkbox' with types Bool, Bool, field: Avram::PermittedAttribute(Bool), class: String`
(It took me a while to figure out that meant the type Lucky was seeing from the form was a type mis-match).

Finally, I decided to use a select form where I could have text for the use and a boolean for Lucky:
```
# src/components/pets/form_fields.cr
class Pets::FormFields < BaseComponent
  needs operation : SavePet

  def render
    mount Shared::Field, operation.name, &.text_input(autofocus: "true")
    mount Shared::Field, operation.breed
    mount Shared::Field, operation.species
    mount Shared::Field, operation.age

    # we are using a select with an array of tuples - human readable words and boolean for lucky:
    # [{"yes", true}, {"no", false}]
    mount Shared::Field, operation.house_trained do |input_html|
      input_html.select_input append_class: "select-input" do
        options_for_select operation.house_trained, [{"yes", true}, {"no", false}]
      end
    end

    # human name for the user to see and the human's db_id for Lucky
    mount Shared::Field, operation.human_id do |input_html|
      input_html.select_input append_class: "select-input" do
        options_for_select operation.human_id, options_for_humans
      end
    end
  end

  # create an array of tuples with the human_name and human id - ie: [{"bill", 1}, {"jane", 2}]
  private def options_for_humans
    OwnerQuery.new.map do |human|
      { human.name, human.id }
    end
  end
end
```

Fixing the age input as a number was an easy casting fix with: `mount Shared::Field, operation.age, &.number_input(attrs: [:required], append_class: "custom-input", min: "0", max: "99")` - this restricts the pets age range which seemed reasonable.

I also figured we would do a dropdown for species (not breeds - there are way to many animal breeds) - since I only have had cats and dogs I used:
```
    mount Shared::Field, operation.species do |input_html|
      input_html.select_input append_class: "select-input" do
        options_for_select operation.species, [{"Dog", "dog"}, {"Cat", "cat"}]
      end
    end
```

So now the completed form looks like:
```
# src/components/pets/form_fields.cr
class Pets::FormFields < BaseComponent
  needs operation : SavePet

  def render
    mount Shared::Field, operation.name, &.text_input(autofocus: "true", attrs: [:required])
    mount Shared::Field, operation.breed, &.text_input(attrs: [:required])
    mount Shared::Field, operation.species do |input_html|
      input_html.select_input append_class: "select-input" do
        options_for_select operation.species, [{"Dog", "dog"}, {"Cat", "cat"}]
      end
    end
    mount Shared::Field, operation.age, &.number_input(append_class: "custom-input", min: "0", max: "99")
    mount Shared::Field, operation.house_trained do |input_html|
      input_html.select_input append_class: "select-input" do
        options_for_select operation.house_trained, [{"yes", true}, {"no", false}]
      end
    end
    mount Shared::Field, operation.owner_id do |input_html|
      input_html.select_input append_class: "select-input" do
        options_for_select operation.owner_id, options_for_owners
      end
    end
  end

  private def options_for_owners
    OwnerQuery.new.map do |owner|
      { owner.name, owner.id }
    end
  end

end
```

## Display Validation Errors

If we leave some fields out - Lucky gives us validation errors - all fields appear to be required without explicitly allowing nils - but we don't see them with our default form.  Lets fix that.

## Validations & Optional Fields

Let's figure out how to allow some fields to be nil

## Tailwind & Alpine Integration

Let's make the forms a little nicer
