---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Bridgetown 1.0 (Beta) - Ruby Static-Site Orientation"
subtitle: "Static Site Generation with Ruby"
summary: "A fun and straightforward way to build static sites using your Ruby & Rails knowledge"
authors: ["btihen"]
tags: ['Ruby', 'ERB', 'Static Site', 'Bridgetown']
categories: []
date: 2022-03-05T01:57:00+02:00
lastmod: 2022-03-07T01:57:00+02:00
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

## **Overview**

I have often wanted to build websites using as much of my Rails knowledge as possible.  Now I can!

Enter Bridgetown - https://edge.bridgetownrb.com/docs

A ruby based (erb, components, etc), author-friendly (markdown pages).

The newest version 1.0 beta uses esbuild by default (or webpacker) and has several pre-build deploy configurations and a quick and easy way to install TailwindCSS!

Unfortunately, for some reason I found it a bit hard to assemble the information to create a website that would meet my needs and interests.  So this doc is a quick summary and context that will get you through the next step after `getting started`.  My understanding is now that are 3 added aspects
*  (**bundled configurations**)[https://www.bridgetownrb.com/docs/bundled-configurations] are for `tool-chain setup`, like deployment configs, tailwindcss, stimulusJS, etc.  And
*  **Plugins** are for things that will show-up in the `output html` like SEO Tags, Sitemaps, etc.
*  (**automations**)[https://www.bridgetownrb.com/docs/automations] An automation script is nothing more than a Ruby code file run in the context of an instance
Unfortunately, I still struggle to find the parts I am looking for, so I am adding the links here (to help my future self).

## **New Site Setup**

I listened to the interview about Bridgetown on (Remote Ruby Podcast)[https://remoteruby.transistor.fm/169] so I went and checked it out.  Starting with the (Beta Docs)[https://edge.bridgetownrb.com/docs] - as it has a TailwindCSS installer and lots of excellent deployment setups (in particular Render and Netlify - although I would find Fly.io also interesting)

So I started by downloading the gem:
```bash
gem install bridgetown -N -v 1.0.0.beta3
```

I decided to configure it with the `erb` but you can leave off `-t erb` and use liquid or change erb for serbea templates.  Anyway, I created a new project with `erb` using:
```bash
bridgetown new bridge_tail_site -t erb
cd bridge_tail_site
```

## **Configure TailwindCSS**

Installing TailwindCSS it was straightforward - once I found the right area. Follow the instructions at https://www.bridgetownrb.com/docs/bundled-configurations#tailwindcss.
```bash
bundle exec bridgetown configure tailwindcss
```

I wanted to see if this worked by starting bridgetown with:
```bash
bin/bridgetown start
open localhost:4000
```

## **Configure AlpineJS**

It looked good - so I went on to install AlpineJS (using the embedded script method) at https://alpinejs.dev/essentials/installation - so I went to `src/_partials/_head.erb` and added `<script defer src="https://unpkg.com/alpinejs@3.9.0/dist/cdn.min.js"></script>` just before the `live_reload_dev_js` tag:
```html
<!-- src/_partials/_head.erb -->
...
<!-- AlpineJS script tag-->
<script defer src="https://unpkg.com/alpinejs@3.9.0/dist/cdn.min.js"></script>
<%= live_reload_dev_js %>
```

Then I went to the page `src/_components/shared/navbar.erb` to add an example from (AlpineJS Start-here page)[https://alpinejs.dev/start-here]
```html
<div x-data="{ count: 0 }">
 <button x-on:click="count++"
         class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
   Increment
 </button>
 <span x-text="count"></span>
</div>
```

Cool this works!  So I went and created my navbar and footer.

## **Adding a Custom Font (in CSS)**

We will add the `handlee` font as it is distinctive and easy to see that it works (or not).
Let's get it from (Google Webfonts Helper)[https://google-webfonts-helper.herokuapp.com/fonts/handlee?subsets=latin] site.  **This is a convenient site as it has both the font and the CSS needed.**

Now that you've downloaded the font, create a new folder in the frontend folder and copy the font into it:
```bash
mkdir -p frontend/fonts/handlee
cp ~/Downloads/handlee-v12-latin/* frontend/fonts/handlee/.
```

Now grab the CSS from the Google Webfonts Helper site and copy it into the `frontend/styles/index.css` file (I like to put the font css just below the tailwind imports). So the start of index.css looks like:

```css
/* frontend/styles/index.css */

/* triggers frontend rebuilds */
@import "jit-refresh.css";

/* Set up Tailwind imports */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Import Fonts */
@font-face {
  font-family: 'Handlee';
  font-style: normal;
  font-weight: 400;
  src: local(''),
       url('../fonts/handlee/handlee-v12-latin-regular.woff2') format('woff2'),
       url('../fonts/handlee/handlee-v12-latin-regular.woff') format('woff');
}

/* ... */
```

Now within your CSS definitions you can use: `font-family: 'Handlee';`

Let's try it out - let's add that to the h1 definition in the index.css file - so now that would look like:

```css
/* ... */
h1 {
  margin: 1rem 0 3rem;
  text-align: center;
  font-weight: 900;
  font-size: 2.5rem;
  font-family: 'Handlee';
  color: var(--heading-color);
  line-height: 1.2;
}
/* ... */
```

Be sure the Title of your homepage is now using the 'Handlee' font.

## **Adding a Custom Font (into TailwindCSS)**

Now we need to define this font within TailwindCSS config to have it create a `font-handlee` class so we can use this font within our tailwind class definitions.  To do this we will need to update the `tailwind.config.js` file to look like:
```js
module.exports = {
  content: [
    './src/**/*.{html,md,liquid,erb,serb}',
    './frontend/javascript/**/*.js',
  ],
  theme: {
    extend: {
      fontFamily: {
        handlee: ['Handlee']
      },
    },
  },
  plugins: [],
}
```

Let's update the default layout to use Handlee for the text within the main body. So lets open `src/_layouts/default.erb` and change the `main` tag to have the `class="font-handlee"` in it - so now it might look like:
```html
<!-- ... -->
    <main class="font-hand">
      <%= yield %>
    </main>
<!-- ... -->
```

Now both the Title and Body of each page should be using the Handlee font.

## **Adding a new Top-Level Page**

Let's add a `contact` page:
```ruby
mkdir src/_pages
cat <<EOF>> src/_pages/contact.md
---
layout: page
title: Contact
---

<h1>Contact Me</h1>
EOF
```

Now if you go to: http://localhost:4000/contact you should see your new page.

For tidiness I prefer to have:

* index.md
* posts.md
* about.md

all in the `src/_pages` folder

## **Adding an Image**

So to add an image we need to put it in the `src/images` folder:
```bash
mkdir -p /images/posts/welcome_post
cp ~/Desktop/sunrise.jpeg /images/posts/welcome_post/.
```

Now let's test this in our **navbar** file `src/_components/shared/navbar.erb`:
```html
<header>
  <%# <img src="/images/logo.svg" alt="Logo" /> %>
  <img src="/images/posts/welcome/sunrise.jpeg" alt="Sunrise" />
</header>
<!-- ... -->
```

Bridgetown uses **Kramdown** as the Markdown rendering engine.  You can learn more about Kramdown Markdown at: https://kramdown.gettalong.org/quickref.html

Let's also add it in our sample blog post `src/_posts/2022-03-05-welcome-to-bridgetown.md`:
```markdown
---
layout: post
title:  "Your First Post on Bridgetown"
date:   2022-03-05 23:22:30 +0100
categories: updates
---
**Display our image!**

![Sunrise](/images/posts/welcome/sunrise.jpeg)

_Now on to the post_
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `bridgetown serve`, which launches a web server and auto-regenerates your site when a file is updated.

...
```

Hopefully you see the image:
* once on the page `http://localhost:4000/`
* twice on the page `http://localhost:4000/updates/2022/03/05/welcome-to-bridgetown/`

## **New post**

A new page within a dated collection.

We just need to make a new file with the correct headers.
```
touch src/_posts/playing_with_bridgetown.md
cat <<EOF>>src/_posts/playing_with_bridgetown.md
---
layout: post
title:  "Fun with Bridgetown"
date:   2022-03-07 01:01:01 +0100
categories: playing
---

## Fun is Rewarding
EOF
```

Now if you go to: http://localhost:4000/posts your new page's title should be listed and if you click on it's title you should see the page with the URL: http://localhost:4000/playing/2022/03/07/playing_with_bridgetown/ - the `category` is the first part of the url, then the date, and finally the title.

## **Define a New Collections**

coming soon



## **New Collection with Custom URLs**

Lets make a news file:
```markdown
mkdir src/news
touch src/news/breaking.md
---
layout: page
title: Breaking
date: 2022-03-07
---

**Breaking News**
EOF
```
If go to: `localhost:4000/news/breaking` we should see our page.


Now lets make an index page that collects all the 'news' files.
```markdown
touch src/news/index.md
cat <<EOF>>src/news/index.md
---
layout: page
title: News
date: 2022-03-07
---

**Some articles**

<% full_path, current_page = File.split(page.path) %>
<% path_list = Dir[full_path + "/*.md"].reject { |p| p.include?(current_page) } %>
<% file_list = path_list.map { |p| File.split(p).last.split('.').first } %>
<% title_list = path_list.map { |f| File.read(f).split("\n").detect {|t| t.include?('title: ') }.split.last } %>
<% link_list = title_list.zip(file_list) %>

<ul>
  <% link_list.each do |title, file| %>
    <li><a href="<%= file %>"><%= title %>- <%= file %></a></li>
  <% end %>
</ul>
EOF
```

now if you go to http://localhost:4000/news

you should see your new page with the breaking news listed too.

Granted this is a big hack - but works.  I am guessing the bridgetown-routes will help with custom routes, but it didn't work for me (yet).


## **Deploy**

Let's now deploy this Webpage (using the `configure` command) it is very straightforward!

1. First, be sure you have pushed your project to github or gitlab - create the repo online and push it with:
```bash
git add .
git commit -m "Configured w TailwindCSS and Handlee Font"
git remote add origin git@github.com:gitusername/bridge_tail_site.git
git branch -M main
git push -u origin main
```
1. Second, install the config for your deploy service (in this case `netlify`) by typing:
```bash
bundle exec bridgetown configure netlify
git add bin/netlify.sh netlify.toml
git commit -m "add netlify config"
git push
```
3. Third, connect your netlify account to the repo you just created.
4. Four, click `deploy` within the netlify site (if it hasn't already startet) and wait 5-10 mins (yes its kinda slow to deploy) and you should have your new website!

Woo Hoo.

## What didn't work (yet!)

#### Change route names

To replace my existing site I am determined to keep the page paths the same as the existing site.  I haven't figured that out for collections (such as blog posts).  I am not interested in breaking links and search index references - so figuring this out is critical.

Probably, hopefully, `Bridgetown File Routing` will address this

#### Bridgetown File Routing

Lets try the new File Routing feature described at: https://edge.bridgetownrb.com/docs/routes

First update the `Gemfile` - uncomment: `gem "bridgetown-routes", "~> 1.0.0.beta3", group: :bridgetown_plugins` - now it should look similar to:
```ruby
# Gemfile
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

gem "bridgetown", "~> 1.0.0.beta3"

# Uncomment to add file-based dynamic routing to your project:
gem "bridgetown-routes", "~> 1.0.0.beta3", group: :bridgetown_plugins

gem "puma", "~> 5.5"
```

Now we need to run bundler:
```bash
bundle install
```

Now setup the Roda config `server/roda_app.rb`:
```ruby
# server/roda_app.rb
require "bridgetown-routes"

class RodaApp < Bridgetown::Rack::Roda
  # Uncomment to use Bridgetown SSR:
  # plugin :bridgetown_ssr

  # And optionally file-based routing:
  plugin :bridgetown_routes

  route do |r|
    # Load Roda routes in server/routes (and src/_routes via `bridgetown-routes`)
    Bridgetown::Rack::Routes.start! self
  end
end
```

Now lets add:
```ruby
# ./server/routes/preview.rb

class Routes::Preview < Bridgetown::Rack::Routes
  route do |r|
    r.on "preview" do
      # Our special rendering pathway to preview a page
      # route: /preview/:collection/:path
      r.get String, String do |collection, path|
        item = Bridgetown::Model::Base.find("repo://#{collection}/#{path}")

        unless item.content.present?
          next Bridgetown::Model::Base.find("repo://pages/_pages/404.html")
            .render_as_resource
            .output
        end

        item
          .render_as_resource
          .output
      end
    end
  end
end
```

Now lets make an index page for this route:
```ruby
mkdir -p src/_routes/items
cat <<EOF>src/_routes/items/index.erb
---<%
# route: /items
r.get do
  render_with data: {
    layout: :page,
    title: "Dynamic Items",
    items: [
      { number: 1, slug: "123-abc" },
      { number: 2, slug: "456-def" },
      { number: 3, slug: "789-xyz" },
    ]
  }
end
%>---

<ul>
  <% resource.data.items.each do |item| %>
    <li><a href="/items/<%= item[:slug] %>">Item #<%= item[:number] %></a></li>
  <% end %>
</ul>
EOF
```

Now lets create the template for items:
```ruby
cat <<EOF>>src/_routes/items/[slug].erb
---<%
# route: /items/:slug
r.get do
  item_id, *item_sku = r.params[:slug].split("-")
  item_sku = item_sku.join("-")

  render_with data: {
    layout: :page,
    title: "Item Page",
    item_id: item_id,
    item_sku: item_sku
  }
end
%>---

<p><strong>Item ID:</strong> <%= resource.data.item_id %></p>

<p><strong>Item SKU:</strong> <%= resource.data.item_sku %></p>
EOF
```

#### AlpineJS installed as a module

Not a show stopper but irritates me.

I tried using the Bridgetown Javascript install instructions at: https://www.bridgetownrb.com/docs/frontend-assets#javascript & also the AlpineJS instructions at: https://alpinejs.dev/essentials/installation#as-a-module

I am hoping to install AlpineJS as an imported module (so building isn't depending on a web-connection and the code needed is local).  So I tried removing the AplineJS script tag from the header:
```html
<!-- src/_partials/_head.erb -->
...
<!-- AlpineJS script tag-->
<%# <script defer src="https://unpkg.com/alpinejs@3.9.0/dist/cdn.min.js"></script> %>
<%= live_reload_dev_js %>
```

Then installing alpinejs
```bash
yarn add alpinejs
```
and I confirmed that I see the AlpineJS in the node_modules folder.

Then I import and start AlpineJS in `frontend/javascript/index.js` so it looked like:
```javascript
// frontend/javascript/index.js
import "index.css"
import Alpine from 'alpinejs'
// Import all JavaScript & CSS files from src/_components
import components from "bridgetownComponents/**/*.{js,jsx,js.rb,css}"

console.info("Bridgetown is loaded!")
window.Alpine = Alpine
Alpine.start()
```


But unfortunately, this doesn't work :( If you know how to make it work, I'll be glad to update this document.

I have also created a github discussion to hopefully help: https://github.com/bridgetownrb/bridgetown/discussions/506

## Feature still to explore

**Add a Custom Font**


**Bundle Configs**
* Setup for purging css: (bundle exec bridgetown configure purgecss) - https://www.bridgetownrb.com/docs/bundled-configurations#purgecss-post-build-hook - installed by default with Tailwind
* Rails Default JS - (bundle exec bridgetown configure stimulus) - https://www.bridgetownrb.com/docs/bundled-configurations#stimulus
* Rails Turbo features: (bundle exec bridgetown configure turbo) - https://www.bridgetownrb.com/docs/bundled-configurations#turbo
* Animation Transitions: (bundle exec bridgetown configure swup) - https://www.bridgetownrb.com/docs/bundled-configurations#swupjs-page-transitions

**Automations**
* Bulma Configured Site: (bundle exec bridgetown apply https://github.com/whitefusionhq/bulmatown)
* Cloudinary Configuration: (bundle exec bridgetown apply https://github.com/bridgetownrb/bridgetown-cloudinary)
* Netlify Configuration: (bundle exec bridgetown apply https://github.com/bridgetownrb/automations/netlify.rb) - how is this different from Netlify bundle configure?

**Testing**
* MiniTests: (bundle exec bridgetown configure minitesting) - https://www.bridgetownrb.com/docs/testing#use-ruby-and-minitest-to-test-html-directly
* Cypres JS Testing: (bundle exec bridgetown apply https://github.com/ParamagicDev/bridgetown-automation-cypress)

**Plugins**
* SEO Tags (bundle add bridgetown-seo-tag -g bridgetown_plugins): https://github.com/bridgetownrb/bridgetown-seo-tag
* Atom Feed (bundle add bridgetown-feed -g bridgetown_plugins): https://github.com/bridgetownrb/bridgetown-feed
* SVG in HTML inline (bundle add "bridgetown-svg-inliner" -g bridgetown_plugins): https://github.com/ayushn21/bridgetown-svg-inliner
* Liquid QuickSearch (bundle add bridgetown-quick-search -g bridgetown_plugins): https://github.com/bridgetownrb/bridgetown-quick-search
* Add a SiteMap (bundle add bridgetown-sitemap -g bridgetown_plugins): https://github.com/ayushn21/bridgetown-sitemap
* Markdown JS (bundle add bridgetown-mdjs -g bridgetown_plugins): https://github.com/bridgetownrb/bridgetown-mdjs
* HTML Minify (bundle add bridgetown-minify-html -g bridgetown_plugins): https://github.com/bt-rb/bridgetown-minify-html
* Github ViewComponents (bundle add bridgetown-view-component -g bridgetown_plugins): https://github.com/bridgetownrb/bridgetown-view-component -- but the docs are here: https://www.bridgetownrb.com/docs/components/ruby#need-compatibility-with-rails-try-viewcomponent-experimental
* GraphQL Api for Bridgetown (bundle add graphtown -g bridgetown_plugins): https://github.com/whitefusionhq/graphtown
* Bulma Starter (bundle exec bridgetown apply https://github.com/whitefusionhq/bulmatown): https://github.com/whitefusionhq/bulmatown (something went wrong on my first try - and don't use this with tailwindcss :)

**Content Management Plugins**
* Notable MD Editor (bundle add bridgetown-notable -g bridgetown_plugins): https://github.com/jamie/bridgetown-notable
* Prismic Flat CMS (bin/bridgetown apply https://github.com/bridgetownrb/bridgetown-prismic): https://github.com/bridgetownrb/bridgetown-prismic

## Conclusion

This looks promising for people familiar with Rails, we will see how it competes with Astro and the other JAMF Stacks for the general public.

So far, the only downsides have been:
* I am not sure I fully understand the logic of 3 added aspects of additional features - for example why is there a netlify automation and bundle config?
* I have only been able to install AlpineJS as a weblink and not as an included module (If I figure it out I'll update this document and or make a configuration script) - maybe I just need to learn into StimulusJS.
* I would like to use Fly.io too (if I figure it out I'll write a configuration script)

Apparently, Vue, React, Bulma plugin-configuations are comming too.
As well as workflows and deployment for github and gitlab.
This should be interesting and fun.
