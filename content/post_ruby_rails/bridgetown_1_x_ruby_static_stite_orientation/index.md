---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Bridgetown 1.0 (Beta) - Ruby Static-Site Orientation"
subtitle: "Static Site Generation with Ruby"
summary: "A fun and straightforward way to build static sites using your Ruby & Rails knowledge"
authors: ["btihen"]
tags: ['Ruby', 'ERB', 'Static Site', 'Bridgetown']
categories: []
date: 2022-03-06T01:57:00+02:00
lastmod: 2022-03-06T01:57:00+02:00
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
bridgetown new tailwind_site -t erb
cd tailwind_site
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

## **Deploy**

1. First, install the config for your deploy service:
```bash
bundle exec bridgetown configure netlify
git add bin/netlify.sh netlify.toml
git commit -m "add netlify config"
git push
```
2. Second, create and push your repo to github or gitlab
3. Third, connect your netlify account to the repo
4. Four, click deploy and wait 5-10 mins and you should have your new website - just click on preview :)


## What didn't work (yet!)

I am hoping to install AlpineJS as an imported module (so building isn't depending on a webconnection and the code needed is local).  So I tried removing the AplineJS script tag from the header:
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

## Feature still to explore

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
