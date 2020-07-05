---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Phoenix_1_5_blog_w_comments"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-07-10T09:43:51+02:00
lastmod: 2020-07-10T09:43:51+02:00
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
## Purpose

This article creates a basic web application backed by a database and creates a few relationships.  I'll use the mix generator commands to make this process quick and easy.  In step two we will add a graphql api.

find the most recent phoenix version:
https://github.com/phoenixframework/phoenix/releases

## Getting Started - create an app

```
mix archive.install hex phx_new 1.5.3
mix phx.new feenix_intro
cd feenix_intro
mix ecto.create
```

test with: `mix phx.server` and go to `http://localhost:4000`

Ideally you see a the Phoenix Start Page.

Let's create a git snapshot
```
git init && git add -A && git commit -m "init"
```

## Lets create two contexts for our Blogs and Accounts

Blogs will have the posts and comments and Accounts will have the user and login credentials and user relationships (why not)?  To see the full documentation on Contexts see: https://hexdocs.pm/phoenix/contexts.html

We will generate two resources and Contexts (and add more later) - lets start with users who will post their blogs (users will be within the Accounts context and posts will be within the Blogs context):
```
mix phx.gen.html Accounts User users name:string email:string username:string:unique
mix phx.gen.html Blogs Post posts title:string body:text user_id:references:users
```

Notice we can generate unique fields with `:unique`

And we can generate relationships (foriegn keys) with `references`


Now that we have generated our code - we need to make a few updates:

First: we need to update our routes in the scope area (`lib/ideas_web/router.ex`) to look like:
```
  scope "/", FeenixIntroWeb do
    pipe_through :browser

    get "/", PageController, :index
    resources "/users", UserController
    resources "/posts", PostController
  end
```

Second before we migrate we need to define the relationships:

so we update the users with a has_many relationship to posts
```
# lib/feenix_intro/accounts/user.ex
defmodule FeenixIntro.Accounts.User do
  use Ecto.Schema
  import Ecto.Changeset
  alias FeenixIntro.Blogs.Post

  @required_fields [:name, :email, :username]

  schema "users" do
    has_many(:posts, Post)

    field :name, :string
    field :email, :string
    field :username, :string

    timestamps()
  end

  @doc false
  def changeset(user, attrs) do
    user
    |> cast(attrs, @required_fields)
    |> validate_required(@required_fields)
    |> unique_constraint(:username)
  end
end
```
If you skip the alias, then `has_many` needs to be written as: `has_many(:posts, FeenixIntro.Blogs.Post)`

Also update post -- REPLACE the `field :user_id, :id` with `belongs_to(:user, User)` -- you CAN'T have both!
```
# lib/feenix_intro/blogs/post.ex
defmodule FeenixIntro.Blogs.Post do
  use Ecto.Schema
  import Ecto.Changeset
  alias FeenixIntro.Blogs.Post
  alias FeenixIntro.Accounts.User

  @required_fields [:user_id, :title, :body]

  schema "posts" do
    belongs_to(:user, User)

    # field :user_id, :id
    field :body, :string
    field :title, :string

    timestamps()
  end

  @doc false
  def changeset(post, attrs) do
    post
    |> cast(attrs, @required_fields)
    |> validate_required(@required_fields)
  end
end
```
NOTE: `@required_fields [:user_id, :title, :body]` isn't required, but as things change defining a constant that can be reused can be convient.

Finally, to be sure we don't have unreferenced blogs if a user gets deleted we need to change our Blog migration to:
```
# priv/repo/migrations/20200704152318_create_posts.exs
defmodule FeenixIntro.Repo.Migrations.CreatePosts do
  use Ecto.Migration

  def change do
    create table(:posts) do
      add :title, :string
      add :body, :text
      # add :user_id, references(:users, on_delete: :nothing)
      add :user_id, references(:users, on_delete: :delete_all), null: false

      timestamps()
    end

    create index(:posts, [:user_id])
  end
end
```

Now it should be safe to migrate using:
```
mix ecto.migrate
```

NOTE: the API's for our Contexts `Accounts` and `Blogs` is in `lib/feenix_intro/accounts.ex` and `lib/feenix_intro/blogs/post.ex` respectively - as we add more info into these contexts these files will get long!  **Ideally you will always interact with the Context API and not the Repo directly this will help create much more managable code.**

Go to: `http://localhost:4000/users`

Now go to: `http://localhost:4000/posts`

notice we can't create a post since we required the user_id and there is not field for that (oops) - we will fix this later for now just create blogs using the seed file:
```
# priv/repo/seeds.exs

# Script for populating the database. You can run it as:
#
#     mix run priv/repo/seeds.exs
#
# We recommend using the bang functions (`insert!`, `update!`
# and so on) as they will fail if something goes wrong.

alias FeenixIntro.Repo
alias FeenixIntro.Blogs.Post
alias FeenixIntro.Accounts.User

# reset the datastore
Repo.delete_all(User) # this should also delete all Posts

# insert people
me = Repo.insert!(%User{ name: "Bill", email: "bill@example.com", username: "bill" })
dog = Repo.insert!(%User{ name: "Nyima", email: "nyima@example.com", username: "nyima" })
Repo.insert!(%Post{ user_id: me.id, title: "Elixir", body: "Very cool ideas" })
Repo.insert!(%Post{ user_id: me.id, title: "Phoenix", body: "live is fascinating" })
Repo.insert!(%Post{ user_id: dog.id, title: "Walk", body: "oh cool" })
Repo.insert!(%Post{ user_id: dog.id, title: "Dinner", body: "YES!" })
```

now as the comments state run: `mix run priv/repo/seeds.exs`

when we list users and create users - all is well

when we do the same withe posts - we get an error creating new posts and we don't see the author in index and show

**Fix Author display**

TODO

**To create or edit a post we need to make the following updates**

In the controller we must load users and add the user_id to the post form:
whe we look in the Accounts API we see: `list_users()`
```
# lib/feenix_intro_web/controllers/post_controller.ex
  # ...
  # add the accounts context alias
  alias FeenixIntro.Accounts
  # ...
  def new(conn, _params) do
    # get users from the Accounts context
    users = Accounts.list_users()
    changeset = Blogs.change_post(%Post{})
    render(conn, "new.html", changeset: changeset, users: users)
    # users: users - sends the users to the form as @users
  end
  # ...
  def edit(conn, %{"id" => id}) do
    post = Blogs.get_post!(id)
    users = Accounts.list_users()
    changeset = Blogs.change_post(post)
    render(conn, "edit.html", post: post, changeset: changeset, users: users)
  end
# ...
```

Now we need to adapt the form to give us a choice of users:
```
# lib/feenix_intro_web/templates/post/form.html.eex
<%= form_for @changeset, @action, fn f -> %>
  <%= if @changeset.action do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below.</p>
    </div>
  <% end %>

  <%= label f, "Author" %>
  <%= select f, :user_id, Enum.map(@users, &{&1.name, &1.id}) %>
  <%= error_tag f, :user %>
  # ...
```

Assuming you can create posts now, lets display the Blog author - that's often interesting to others.
We can do this with preloading in our Blog context:
```
# lib/feenix_intro/blogs.ex
  # change this line:
  # def list_posts, do: Repo.all(Post)
  def list_posts do
    Post
    |> Repo.all()
    |> Repo.preload(:user)
  end
  ```
and also our get_post
```
# lib/feenix_intro/blogs.ex
  # change:
  # def get_post!(id), do: Repo.get!(Post, id)
  # into:
  def get_post!(id) do
    Post
    |> Repo.get!(id)
    |> Repo.preload(:user)
  end
```
now we can update our index and show page to display the author's name at the top of the page:
```
# lib/feenix_intro_web/templates/post/show.html.eex
<h1>Show Post</h1>

<ul>

  <li>
    <strong>Author:</strong>
    <%= @post.user.name %>
  </li>
```
and in the index too:
```
# lib/feenix_intro_web/templates/post/index.html.eex
# ...
<%= for post <- @posts do %>
    <tr>
      <td><%= post.user.name %></td>
      <td><%= post.title %></td>
      <td><%= post.body %></td>
# ...
```


Now that we can see the Post author, lets make another git snapshot:
```
git add .
git commit -m "users and posts resources are created and working independenatly"
```


## now lets create comments (a has many through for users)

we will use `mix phx.gen.context` this time since we will use the posts page to add comments

```
mix phx.gen.context Blogs Comment comments message:text post_id:references:posts  user_id:references:users
```

Again we need to create the relationships and update the migration to delete comments when post is deleted:

Lets start with the migration (dependent delete):
```
# priv/repo/migrations/20200704161651_create_comments.exs
      # ...

      # replce
      # add :post_id, references(:posts, on_delete: :nothing)
      # add :user_id, references(:users, on_delete: :nothing)

      # with
      add :post_id, references(:posts, on_delete: :delete_all), null: false
      add :user_id, references(:users, on_delete: :delete_all), null: false

      # ...
```

Now lets create the relationship between posts and comments:
```
# lib/feenix_intro/blogs/comment.ex
defmodule FeenixIntro.Blogs.Comment do
  use Ecto.Schema
  import Ecto.Changeset
  # for simplicity add the aliases
  alias FeenixIntro.Blogs.Post
  alias FeenixIntro.Accounts.User

  @required_fields [:user_id, :post_id, :message]

  schema "comments" do
    # REMOVE THESE!
    # field :post_id, :id
    # field :user_id, :id

    belongs_to(:user, User)
    belongs_to(:post, Post)
    field :message, :string

    timestamps()
  end

  @doc false
  def changeset(comment, attrs) do
    comment
    |> cast(attrs, @required_fields)
    |> validate_required(@required_fields)
  end
end
```

Now lets update posts relationship to comments:
```
# lib/feenix_intro/blogs/post.ex
  # ...
  # add comment alias
  alias FeenixIntro.Blogs.Comment

  # ...

  schema "posts" do
    belongs_to(:user, User)

    # add a has many comments
    has_many(:comments, Comment)

  # ...
```

# context has no webstuff
mix phx.gen.html Blogs Post posts title:string body:text user_id:references:users



mix phx.gen.context Account Friendship friendships user_id:references:users friend_id:references:users

lib/ideas/account/friendship.ex
defmodule Ideas.Account.Friendship do
  use Ecto.Schema
  import Ecto.Changeset
  alias Ideas.Account.User
  alias Ideas.Account.Friendship

  @required_fields [:user_id, :friend_id]

  schema "friendships" do
    belongs_to(:user, User)
    belongs_to(:friend, User)

    # field :user_id, :id
    # field :friend_id, :id

    timestamps()
  end

  @doc false
  # def changeset(%Friendship{} = friendship, attrs) do
  def changeset(friendship, attrs) do
    friendship
    |> cast(attrs, @required_fields)
    |> validate_required(@required_fields)
  end
end

also update: lib/ideas/account/user.ex
defmodule Ideas.Account.User do
  use Ecto.Schema
  import Ecto.Changeset
  alias Ideas.Blogs.Post
  alias Ideas.Accounts.User
  alias Ideas.Accounts.Friendship

  schema "users" do
    field :email, :string
    field :name, :string
    field :username, :string

    has_many(:posts, Post)
    has_many(:friendships, Friendship)
    has_many(:friends, through: [:friendships, :friend])

    timestamps()
  end

  @doc false
  def changeset(user, attrs) do
    user
    |> cast(attrs, [:name, :email, :username])
    |> validate_required([:name, :email, :username])
  end
end


Now update the seeds:
```
# Script for populating the database. You can run it as:
#
#     mix run priv/repo/seeds.exs
#
# Inside the script, you can read and write to any of your
# repositories directly:
#
#     Ideas.Repo.insert!(%Ideas.SomeSchema{})
#
# We recommend using the bang functions (`insert!`, `update!`
# and so on) as they will fail if something goes wrong.
alias Ideas.Repo
alias Ideas.Blog.Post
alias Ideas.Account.User
alias Ideas.Account.Friendship

# reset the datastore
Repo.delete_all(User)

# insert people
me = Repo.insert!(%User{ name: "Bill Tihen", email: "btihen@gmail.com", username: "btihen" })
Repo.insert!(%Post{ user_id: me.id, title: "Elixir", body: "Very cool ideas" })
Repo.insert!(%Post{ user_id: me.id, title: "Phoenix", body: "live is fascinating" })

ct = Repo.insert!(%User{ name: "Conrad Taylor", email: "conradwt@gmail.com", username: "conradwt" })
Repo.insert!(%Post{ user_id: ct.id, title: "Phoenix-GraphQL", body: "very helpful stuff" })

dhh = Repo.insert!(%User{ name: "David Heinemeier Hansson", email: "dhh@37signals.com", username: "dhh" })
ezra = Repo.insert!(%User{ name: "Ezra Zygmuntowicz", email: "ezra@merbivore.com", username: "ezra" })
matz = Repo.insert!(%User{ name: "Yukihiro Matsumoto", email: "matz@heroku.com", username: "matz" })

ct
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: ct.id, friend_id: matz.id } )
|> Repo.insert

dhh
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: dhh.id, friend_id: ezra.id } )
|> Repo.insert

dhh
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: dhh.id, friend_id: matz.id } )
|> Repo.insert

ezra
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: ezra.id, friend_id: dhh.id } )
|> Repo.insert

ezra
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: ezra.id, friend_id: matz.id } )
|> Repo.insert

matz
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: matz.id, friend_id: ct.id } )
|> Repo.insert

matz
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: matz.id, friend_id: ezra.id } )
|> Repo.insert

matz
|> Ecto.build_assoc(:friendships)
|> Friendship.changeset( %{ user_id: matz.id, friend_id: dhh.id } )
|> Repo.insert
```