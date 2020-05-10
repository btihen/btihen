---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Phoenix 1.5 LiveView Basics"
subtitle: "A Websocket Counter -- without JavaScript"
summary: "Create a simple web counter app to learn how Phoenix 1.5 LiveView works.  Phoenix LiveView allows dynamic webpages with fast update times -- without JavaScript."
authors: ["Bill Tihen"]
tags: []
categories: []
date: 2020-05-10T17:01:53+02:00
lastmod: 2020-05-10T17:01:53+02:00
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
I have been watching Phoenix and Elixir for a while, but the idea of writing dynamic Web Applications without needing a ton of JavaScript is very interesting.  I recently saw this video:

* https://www.youtube.com/watch?v=MZvmYaFkNJI&feature=youtu.be


which is very cool, but I wanted to understand how things actually work -- So I found a Phoenix 1.4 tutorial (without using the liveview generators since it wasn't so tightly integrated into Phoenix at that time) and would translate that into Phoenix 1.5. Here is the video and blog I used to guide me:

* https://www.youtube.com/watch?v=2bipVjOcvdI
* https://dennisbeatty.com/2019/03/19/how-to-create-a-counter-with-phoenix-live-view.html

Since I am just learning the Phoenix Framework and will need to refer to this for my self to remember how to do basic things -- I've documented every little detail.

## Step 0 - Setup

Setup environment & newest version of elixir (I am using exenv):

```
exenv install 1.10.3
exenv global
exenv local 1.10.3
```

Install the 1.5.1 phx_new generator:

`mix archive.install hex phx_new 1.5.1`

Create the project:

`mix phx.new counter --no-ecto --live`

enter project and create init commit:

`
cd counter
`

Now we will run the auto generated test to be sure all is good:

`mix text`

Now we will start phoenix to be sure all is running:

`mix phx.server`

Now go to the web page it is server (probably): `localhost:4000`

It all looks good so we can make our first commit.

`git init && git add -A && git commit -m "init"`

## Step 1 - simple counter page

Make a counter_live folder & an index.ex file:
```
mkdir lib/counter_web/live/counter_live
touch lib/counter_web/live/counter_live/index.ex
```

add the following contents:
```
cat <<"EOF"> lib/counter_web/live/counter_live/index.ex
defmodule CounterWeb.CounterLive.Index do
  use CounterWeb, :live_view
  # use Phoenix.LiveView

  # since we don't have a db to pull from we initialize on mount
  @impl true
  def mount(_params, _session, socket) do
    {:ok, assign(socket, :val, 0)}
  end

  def render(assigns) do
    ~L"""
      <h1>Live Counter</h1>
      <p>
        <b>Here is a great complex page</b>
      </p>
      <div>
        <h2>The count is: <%= @val %></h2>
        <button phx-click="dec">-</button>
        <button phx-click="inc">+</button>
      </div>
      <div>
        <button phx-click="clear">clear</button>
      </div>
      <p>
        <i>even more awesome content</i>
      </p>
    """
  end

  def handle_event("inc", _, socket) do
    {:noreply, update(socket, :val, &(&1 + 1))}
  end

  def handle_event("dec", _, socket) do
    {:noreply, update(socket, :val, &(&1 - 1))}
  end

  def handle_event("clear", _, socket) do
    {:noreply, update(socket, :val, &(&1 - &1))}
    # {:noreply, update(socket, :val, 0)} # very slow - why?
  end

end
EOF
```
Notice: `phx-click="clear"` refers to a `handle_event` which is similar to JavaScript events (but SERVER SIDE)! :)


Now update the routers and it should work:
```
  scope "/", CounterWeb do
    pipe_through :browser

    # live "/", PageLive, :index        # remove this line
    live "/", CounterLive.Index, :index # add this line
  end
```

now start pheonix:

`mix phx.server`

now to:

`localhost:4000`

it works - we have our counter.

Lets run the test:

`mix test`

Opps - we need to fix the homepage test:

since we replaced the original route we need to delete the existing LivePageTest and create a new test for our counter page:

```
rm test/counter_web/live/page_live_text.exs

touch test/counter_web/live/counter_live_text.exs
cat <<"EOF" > test/counter_web/live/counter_live_text.exs
defmodule CounterWeb.CounterLiveTest do
  use CounterWeb.ConnCase

  import CounterWeb.CounterLive.Index

  test "disconnected and connected render", %{conn: conn} do
    {:ok, page_live, disconnected_html} = live(conn, "/")
    assert disconnected_html =~ "Live Counter"
    assert render(page_live) =~ "Live Counter"
  end

end
EOF
```

Now we can test again `mix test`

cool - all is green  _(Ideally - I hopefully one can test the live view actions -- without cucumber - but I don't know how yet.)_

I'll take a git snapshot:

```
git add .
git commit -m "counter with live update"
```

## Step 2 -- us a template with live_view

create a template file (helpful for complex html pages, but simple to create):

`touch lib/counter_web/live/counter_live/index.html.leex`

now just copy the html (from the render method into this file):

```
cat <<"EOF" > lib/counter_web/live/counter_live/index.html.leex
<h1>Live Counter</h1>

<p>
  <b>Here is a great complex page</b>
</p>

<div>
  <h2>The count is: <%= @val %></h2>
  <button phx-click="dec">-</button>
  <button phx-click="inc">+</button>
</div>
<div>
   <button phx-click="clear">clear</button>
</div>

<p>
  <i>even more awesome content</i>
</p>
EOF
```

now point `lib/counter_web/live/counter_live/index.ex` to this file by replacing render with an apply command:

```
  defp apply_action(socket, :index, _params) do
    socket
  end
  #  def render(assigns) do
  #  ~L"""
  #    <h1>Live Counter</h1>
  #    <p>
  #      <b>Here is a great complex page</b>
  #    </p>
  #    <div>
  #      <h2>The count is: <%= @val %></h2>
  #      <button phx-click="dec">-</button>
  #      <button phx-click="inc">+</button>
  #    </div>
  #    <div>
  #      <button phx-click="clear">clear</button>
  #    </div>
  #    <p>
  #      <i>even more awesome content</i>
  #    </p>
  #  """
  # end
```

apply_action understand the rest commands such as :new, etc.

THIS ALLOWS a nice clean separation of code logic from the HTML template (or JSON, etc).

now try the app again and it should still work!

and test again: `mix test`

now git snapshot:

```
git add .
git commit -m "counter using a template"
```


# step 3 - Using Reusable Live Components

create a file for the component:

```
touch lib/counter_web/live/counter_live/counter_component.ex
```

Now you can move the dynamic html and it's associated functions into this file:

```
cat <<"EOF" > lib/counter_web/live/counter_live/counter_component.ex
# lib/counter_web/live/counter_live/counter_component.ex
defmodule CounterWeb.CounterLive.CounterComponent do
  use CounterWeb, :live_component

  def render(assigns) do
    ~L"""
    <div>
      <h2>The count is: <%= @val %></h2>
      <button phx-click="dec" phx-target="<%= @myself %>">-</button>
      <button phx-click="inc" phx-target="<%= @myself %>">+</button>
    </div>
    <div>
      <button phx-click="clear" phx-target="<%= @myself %>">clear</button>
    </div>
    """
  end

  def handle_event("inc", _, socket) do
    {:noreply, update(socket, :val, &(&1 + 1))}
  end

  def handle_event("dec", _, socket) do
    {:noreply, update(socket, :val, &(&1 - 1))}
  end

  def handle_event("clear", _, socket) do
    # {:noreply, update(socket, :val, 0)} # very slow - why?
    {:noreply, update(socket, :val, &(&1 - &1))}
  end

end
EOF
```

NOTICE: the `@myself` this means that the function is found locally (for even better encapsulation).

Its important to import the live view suff again.

Now update the live template to point at the component:

in `lib/counter_web/live/counter_live/index.ex` REMOVE:
```
<div>
  <h2>The count is: <%= @val %></h2>
  <button phx-click="dec">-</button>
  <button phx-click="inc">+</button>
</div>
<div>
   <button phx-click="clear">clear</button>
</div>
```

AND in `lib/counter_web/live/counter_live/index.ex` REPLACE WITH:
```
<%= live_component @socket, CounterWeb.CounterLive.CounterComponent, id: 0, val: @val %>
```

Now all the dynamic aspects are all encapsulated as a component -- so we can make complex dynamic pages and keep the our headspace needed to update anyone aspect small.

test one last time `mix test`

one last git snapshot:

```
git add .
git commit -m "live pages using isolated components - like JS does"
```
Cool!

I like this framework!
