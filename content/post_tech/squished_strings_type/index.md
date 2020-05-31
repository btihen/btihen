---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Squished String Type (Rails 5/6)"
subtitle: "String inputs that strip away leading, trailing and double spaces using typed virtual attributes"
summary: "String inputs that strip away leading, trailing and double spaces using typed virtual attributes"
authors: ["Bill Tihen"]
tags: ["Rails 5", "Rails 6", "Types", "Virtual Attributes"]
categories: []
date: 2020-05-03T20:11:22+02:00
lastmod: 2020-05-03T20:11:22+02:00
featured: false
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
Lets start by making a sample project (without tests -T):

```bash {linenos=table,hl_lines=1}
rails new task_cards -T
cd task_cards
```

We will use a generator to create the structures we need and just focus clean inputs.
```bash {linenos=table,hl_lines=1}
rails g scaffold Card title:string description:text
```

Now lets make our attribute types that cleanup string inputs (we will put them in the a new folder we will call `types`):
```bash {linenos=table,hl_lines=1}
mkdir app/types
touch app/types/string_stripped_type.rb
touch app/types/text_trimmed_type.rb
```

To make a string type that removes leading, trailing and duplicate spaces we will use the squish method.
{{< highlight ruby "linenos=table" >}}
# app/types/string_squished_type.rb
class StringSquishedType < ActiveRecord::Type::String
  # cast the incomming value for Rails
  def cast(value)
    value.to_s.squish
  end

  # convert the data to what the Database expects
  def serialize(value)
    value.to_s
  end

end
{{< / highlight >}}

Since text may want to have newlines and other `double` spaces we will only remove (`trim`) leading and trailing spaces:
{{< highlight ruby "linenos=table" >}}
# app/types/text_trimmed_type.rb
class TextTrimmedType < ActiveRecord::Type::String
  # cast the incomming value for Rails
  def cast(value)
    value.to_s.strip
  end

  # convert the data to what the Database expects
  def serialize(value)
    value.to_s
  end

end
{{< / highlight >}}

To simplify our code we will define short names for our new types -- in the `config/initializers` folder we will make a new file called `types.rb`:
```bash {linenos=table,hl_lines=1}
touch config/initializers/attribute_types.rb
```

To make a string type that removes leading, trailing and duplicate spaces we will use the squish method.
{{< highlight ruby "linenos=table" >}}
# config/initializers/attribute_types.rb
ActiveRecord::Type.register(:string_stripped, StringSquishedType)
ActiveRecord::Type.register(:text_trimmed,    TextTrimmedType)
{{< / highlight >}}

Now lets add our new virutual data types that we will use in our forms to our model:
{{< highlight ruby "linenos=table" >}}
# app/models/card.rb
class Card < ApplicationRecord
  attribute :title_in,       :string_squished
  # attribute :title_in,       StringSquishedType.new
  attribute :description_in. :text_trimmed,   default: '--'
  # attribute :description_in, TextTrimmedType.new, default: '--'

  validate :title_in, presence: true
end
{{< / highlight >}}
