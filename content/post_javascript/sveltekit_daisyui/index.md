---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "SvelteKit, TailwindCSS and DaisyUI Website"
subtitle: "Building a Static Site / Blog with SvelteKit"
summary: "A straight forward way to build websites with lots of features"
authors: ["btihen"]
tags: ["Svelte", "SvelteKit", "TailwindCSS", "DaisyUI", "Blog", "StaticSite"]
categories: ["Code", "JavaScript", "Website"]
date: 2022-04-16T01:01:53+02:00
lastmod: 2021-04-18T01:01:53+02:00
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

## Intro

I've been looking for a flexible, yet easy to setup and manage website / blog -- which also easily formats and accommodates code!

I have been playing with the combination of SvelteKit, TailwindCSS and DaisyUI, MDsveX (for MD blog pages).

Here's what I've learned so far.

## Base Software (nodejs)

be sure to have a modern version of nodejs

```bash
asdf install nodejs 16.14.2
asdf global nodejs 16.14.2
```

## Create a SvelteKit Site

Install base Sveltekit (still in beta so `@next` is included)

```bash
npm init svelte@next sveltekit_w_daisyui
cd sveltekit_w_daisyui

# install sveltekit packages
npm install

# initialize Git
git init && git add -A && git commit -m "Initial commit"

# starts sveltekit
npm run dev -- --open
```

## Install/Configure Components

we want to install and configure TailwindCSS, DaisyUI and MDsvex (markdown), `svelte-add@latest` installs and configures too!

```bash
# Tailwind and PostCSS
npx svelte-add@latest tailwindcss
npm i

# MD pages
npx svelte-add@latest mdsvex
npm i
```

now install DaisyUI (note `npm i daisyui` didn't work, but adding `-D` did)
```bash
npm install -D daisyui
```

Now to configure `tailwind.config.js` - add daisyui as a plugin at the end of the file:
```js
module.exports = {
  //...
  plugins: [require("daisyui")],
}
```

My config with a theme looks like (I left my experiments - so you can see how easy it is to adjust and play with):
```js
const config = {
  content: ["./src/**/*.{html,js,svelte,ts}"],

  theme: {
    extend: {},
  },
  // http://colorcode.is/ - helpful color site
  daisyui: {
    themes: [
      {
        // corporate adapted light
        btihenl: {
          primary: "#5aa867",
          secondary: "#0080bc",
          accent: "#f1db85",
          neutral: "#3a3a3a",
          "neutral-content":
          "base-100": "#f0f8ff",
          "base-content": "#100f0f",
          info: "#0080bc",
          success: "#5aa867",
          warning: "#d29a2f",
          error: "#b53729",
          "code-bg-color": "#f1db85",
        },
        // business adapted darker
        btihend: {
          primary: "#5aa867",
          secondary: "#0080bc",
          accent: "#f1db85",
          neutral: "#23282E",
          "base-100": "#3a3a3a",
          info: "#0080bc",
          success: "#5aa867",
          warning: "#d29a2f",
          error: "#b53729",
          "code-bg-color": "#f1db85",
        },
      }
    ],
  },
  plugins: [require("daisyui")],
};

module.exports = config;
```

finally install theme-change: https://github.com/saadeghi/theme-change
```bash
npm i theme-change --save
```
Now configure themechanger:
```css
/* src/app.css */
/* Write your global styles here, in PostCSS syntax */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Allow system color */
:root {
  --color-default: #f2f3f4;
}
/* set a default theme - btihenl is my light theme - more later*/
@media (prefers-color-scheme: btihenl) {
  :root {
    --my-color: #252b30;
  }
}

/* basic important css */
h1 {
  margin: 1rem 0 3rem;
  text-align: center;
  font-weight: 800;
  font-size: 2.0rem;
  color: var(--heading-color);
  line-height: 1.2;
}
```

## Update your homepage

All pages will be within `src/routes`

This is at: `src/routes/index.svelte`

The following will make it easy to tweak your theme:

```svelte
<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<div class="badge">neutral</div>
<div class="badge badge-primary">primary</div>
<div class="badge badge-secondary">secondary</div>
<div class="badge badge-accent">accent</div>
<div class="badge badge-ghost">ghost</div>

<div class="alert shadow-lg">
  <div>
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="stroke-info flex-shrink-0 w-6 h-6">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
    </svg>
    <span>Shadow Notice</span>
  </div>
</div>

<div class="alert alert-info shadow-lg">
  <div>
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="stroke-current flex-shrink-0 w-6 h-6">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
    </svg>
    <span>Info notice.</span>
  </div>
</div>

<div class="alert alert-success shadow-lg">
  <div>
    <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current flex-shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
    </svg>
    <span>Suceess Notice</span>
  </div>
</div>

<div class="alert alert-warning shadow-lg">
  <div>
    <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current flex-shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
    </svg>
    <span>Warning: Invalid email address!</span>
  </div>
</div>

<div class="alert alert-error shadow-lg">
  <div>
    <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current flex-shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
    </svg>
    <span>Error! Task failed successfully.</span>
  </div>
</div>
```

## Create an About Page

Create new pages in `src/routes` -- for example an about page:

```bash
touch src/routes/about.svelte
cat <<EOF>>src/routes/about.svelte
<h1>Hi</h1>

<p>This is my about page.</p>
EOF
```

now go to `http://localhost:4000/about` and you should see your new page.

## Create Components

```bash
mkdir -p src/lib/components
touch src/lib/components/Navbar.svelte
touch src/lib/components/Footer.svelte
```

### Navbar

I'll explain / Setup the Themechange in a bit

```svelte
cat <<EOF >>src/lib/components/Navbar.svelte
<script>
  import { onMount } from 'svelte'
  import { themeChange } from 'theme-change'

  // NOTE: the element that is using one of the theme attributes must be in the DOM on mount
  onMount(() => {
    themeChange(false)
    // false parameter is required for svelte
  })
</script>

<navbar>
  <div class="bg-black h-2.5"></div>
  <div class="flex items-center justify-center sticky top-0 z-50 bg-blue-600">

    <div class="navbar w-4/5">

      <div class="navbar-start">
        <div class="dropdown">
          <div tabindex="0" class="btn btn-ghost btn-circle">
            <!-- menu-alt1 - hero-icon -->
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 22 22" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="4" d="M2 6h16M2 12h8m-8 6h16" />
            </svg>
          </div>
          <ul tabindex="0" class="menu menu-compact dropdown-content mt-3 p-2 shadow bg-base-100 rounded-box w-62">
            <li><a href="/">Item 1</a></li>
            <li tabindex="0">
              <a class="justify-between" href="/">
                Parent
                <svg class="fill-current" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                  <path d="M8.59,16.58L13.17,12L8.59,7.41L10,6L16,12L10,18L8.59,16.58Z"/>
                </svg>
              </a>
              <ul class="p-2 shadow bg-base-100">
                <li><a href="/">Submenu 1</a></li>
                <li><a href="/">Submenu 2</a></li>
              </ul>
            </li>
            <li><a href="/">Item 3</a></li>
            <li>
              <div class="form-control">
                <input type="text" placeholder="Search" class="input input-bordered">
              </div>
            </li>
          </ul>
        </div>
      </div>

      <div class="navbar-center">
        <a href="/" class="btn btn-ghost normal-case text-2xl">btihen.dev</a>
      </div>

      <div class="navbar-end space-x-2">
        <!-- sun - heroicon.dev -->
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"></path>
        </svg>
        <!-- https://github.com/saadeghi/theme-change -->
        <input type="checkbox" class="toggle" id="theme-toggle" data-toggle-theme="btihenl,btihend" data-act-class="ACTIVECLASS">
        <!-- moon - heroicon.dev -->
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"></path>
        </svg>
      </div>
    </div>

  </div>
</navbar>
EOF
```

### Footer

```svelte
cat <<EOF >>src/lib/components/Footer.svelte
<footer class="flex items-center justify-center p-4 bg-gray-600 shadow-md">
  <div class="footer w-4/5">
    <div class="items-center grid-flow-col place-self-center md:place-self-start">
      <a href="/" class="btn btn-ghost normal-case text-2xl">btihen.dev</a>
    </div>

    <div class="grid-flow-col gap-4 place-self-center md:justify-self-end">
      <!-- github - cons.getbootstrap.com-->
      <a href="/">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-github" viewBox="0 0 16 16">
          <path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.012 8.012 0 0 0 16 8c0-4.42-3.58-8-8-8z"/>
        </svg>
      </a>
      <!-- linked-in - cons.getbootstrap.com -->
      <a href="/">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-linkedin" viewBox="0 0 16 16">
          <path d="M0 1.146C0 .513.526 0 1.175 0h13.65C15.474 0 16 .513 16 1.146v13.708c0 .633-.526 1.146-1.175 1.146H1.175C.526 16 0 15.487 0 14.854V1.146zm4.943 12.248V6.169H2.542v7.225h2.401zm-1.2-8.212c.837 0 1.358-.554 1.358-1.248-.015-.709-.52-1.248-1.342-1.248-.822 0-1.359.54-1.359 1.248 0 .694.521 1.248 1.327 1.248h.016zm4.908 8.212V9.359c0-.216.016-.432.08-.586.173-.431.568-.878 1.232-.878.869 0 1.216.662 1.216 1.634v3.865h2.401V9.25c0-2.22-1.184-3.252-2.764-3.252-1.274 0-1.845.7-2.165 1.193v.025h-.016a5.54 5.54 0 0 1 .016-.025V6.169h-2.4c.03.678 0 7.225 0 7.225h2.4z"/>
        </svg>
      </a>
      <!-- twitter - cons.getbootstrap.com-->
      <a href="/">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-twitter" viewBox="0 0 16 16">
          <path d="M5.026 15c6.038 0 9.341-5.003 9.341-9.334 0-.14 0-.282-.006-.422A6.685 6.685 0 0 0 16 3.542a6.658 6.658 0 0 1-1.889.518 3.301 3.301 0 0 0 1.447-1.817 6.533 6.533 0 0 1-2.087.793A3.286 3.286 0 0 0 7.875 6.03a9.325 9.325 0 0 1-6.767-3.429 3.289 3.289 0 0 0 1.018 4.382A3.323 3.323 0 0 1 .64 6.575v.045a3.288 3.288 0 0 0 2.632 3.218 3.203 3.203 0 0 1-.865.115 3.23 3.23 0 0 1-.614-.057 3.283 3.283 0 0 0 3.067 2.277A6.588 6.588 0 0 1 .78 13.58a6.32 6.32 0 0 1-.78-.045A9.344 9.344 0 0 0 5.026 15z"/>
        </svg>
      </a>
      <!-- facebook -->
      <!--
      <a href="/">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" class="fill-current">
          <path d="M9 8h-3v4h3v12h5v-12h3.642l.358-4h-4v-1.667c0-.955.192-1.333 1.115-1.333h2.885v-5h-3.808c-3.596 0-5.192 1.583-5.192 4.615v3.385z"></path>
        </svg>
      </a>
      -->
    </div>
  </div>
</footer>
<div class="bg-black flex items-center justify-center h-3.5">
  <p class="text-white text-2xs">Copyright © 2022 - All right reserved</p>
</div>
EOF
```

## Create a Base Layout

To create a layout `__layout.svelte` (so we can add components to each page like navbars, etc)

```svelte
touch src/routes/__layout.svelte
cat <<EOF>>src/routes/__layout.svelte
<script>
  import "../app.css";
  import Navbar from '$lib/components/Navbar.svelte'
  import Footer from '$lib/components/Footer.svelte'
</script>

<Navbar />

<main class="flex items-center justify-center bg-base-100">
  <div class="w-4/5 m-4 px-4 py-4 sm:px-0">
    <slot /> <!-- content goes into slot -->
  </div>
</main>

<Footer />
EOF
```

## Blog setup

https://github.com/spences10/sveltekit-mdsvex-starter-blog/tree/main/src/lib

## Markdown Setup

https://github.com/spences10/sveltekit-mdsvex-starter-blog/tree/main/src/lib

### External Links

https://github.com/rehypejs/rehype-external-links

### Slugs (links to sections in longer documents)

https://github.com/rehypejs/rehype-slug
https://github.com/rehypejs/rehype-autolink-headings




### Relative Images

https://github.com/mattjennings/mdsvex-relative-images

I like folders with images and all content together so i'll install:
```bash
npm install mdsvex-relative-images
```
Setup in `mdsvex.config.js` with:

```js
// mdsvex.config.js
import relativeImages from "mdsvex-relative-images";

export default {
  // ... rest of your config
  remarkPlugins: [relativeImages],
};
```

BE SURE TO RESTART SVELTEKIT to be sure all is good.

```bash
npm run dev --
```
### Front-mater


### Code formatting

```css
/* Write your global styles here, in PostCSS syntax */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --color-default: #f2f3f4;
}
@media (prefers-color-scheme: btihenl) {
  :root {
    --my-color: #252b30;
  }
}

h1 {
  margin: 1rem 0 3rem;
  text-align: center;
  font-weight: 800;
  font-size: 2.0rem;
  color: var(--heading-color);
  line-height: 1.5;
}
h2 {
  margin: 1.5rem 0 0.5rem;
  font-weight: 600;
  font-size: 1.5rem;
  color: var(--heading-color);
  line-height: 1.3;
}
h3 {
  margin: 1.2rem 0 0.5rem;
  font-weight: 500;
  font-size: 1.3rem;
  color: var(--heading-color);
  line-height: 1.1;
}
p {
  margin: 0.5rem 0;
  font-size: 1.0rem;
  color: var(--text-color);
  line-height: 1.15;
}

/* https://github.com/PrismJS/prism-themes/blob/master/themes/prism-atom-dark.css */
/**
 * atom-dark theme for `prism.js`
 * Based on Atom's `atom-dark` theme: https://github.com/atom/atom-dark-syntax
 * @author Joe Gibson (@gibsjose)
 */
code[class*="language-"],
pre[class*="language-"] {
  color: #c5c8c6;
  text-shadow: 0 1px rgba(0, 0, 0, 0.3);
  font-family: Inconsolata, Monaco, Consolas, 'Courier New', Courier, monospace;
  direction: ltr;
  text-align: left;
  white-space: pre;
  word-spacing: normal;
  word-break: normal;
  line-height: 1.5;

  -moz-tab-size: 4;
  -o-tab-size: 4;
  tab-size: 4;

  -webkit-hyphens: none;
  -moz-hyphens: none;
  -ms-hyphens: none;
  hyphens: none;
}

/* Code blocks */
pre[class*="language-"] {
  padding: 1em;
  margin: .5em 0;
  overflow: auto;
  border-radius: 0.5em;
}

:not(pre)>code[class*="language-"],
pre[class*="language-"] {
  background: #1d1f21;
}

/* Inline code */
:not(pre)>code[class*="language-"] {
  padding: .1em;
  border-radius: .3em;
}

.token.comment,
.token.prolog,
.token.doctype,
.token.cdata {
  color: #7C7C7C;
}

.token.punctuation {
  color: #c5c8c6;
}

.namespace {
  opacity: .7;
}

.token.property,
.token.keyword,
.token.tag {
  color: #96CBFE;
}

.token.class-name {
  color: #FFFFB6;
  text-decoration: underline;
}

.token.boolean,
.token.constant {
  color: #99CC99;
}

.token.symbol,
.token.deleted {
  color: #f92672;
}

.token.number {
  color: #FF73FD;
}

.token.selector,
.token.attr-name,
.token.string,
.token.char,
.token.builtin,
.token.inserted {
  color: #A8FF60;
}

.token.variable {
  color: #C6C5FE;
}

.token.operator {
  color: #EDEDED;
}

.token.entity {
  color: #FFFFB6;
  cursor: help;
}

.token.url {
  color: #96CBFE;
}

.language-css .token.string,
.style .token.string {
  color: #87C38A;
}

.token.atrule,
.token.attr-value {
  color: #F9EE98;
}

.token.function {
  color: #DAD085;
}

.token.regex {
  color: #E9C062;
}

.token.important {
  color: #fd971f;
}

.token.important,
.token.bold {
  font-weight: bold;
}

.token.italic {
  font-style: italic;
}
```

### Reading time

## Blog Index Page

## Blog Tag Search / Pages

## Contact page (netlify)

## Deployment (netlify)

## Deployment (render)

## To Solve

* MDsvex CSS
* MDsvex - layouts
* DaisyUI dropdowns don't close with a second click, but on a focus change :(


## Resources

### Svelte Resources

* https://www.youtube.com/watch?v=haKnkk6ds20
* https://joshcollinsworth.com/blog/build-static-sveltekit-markdown-blog

* https://kit.svelte.dev/
* https://megzari.com/blog/about_this_site
* https://svelte.dev/examples#dom-event-forwarding
* https://prismic.io/blog/svelte-sveltekit-tutorial
* https://dev.to/nico_bachner/sveltekit-theme-switch-a58
* https://www.liip.ch/en/blog/sveltekit-and-tailwind-windi-css
* https://svelteland.github.io/svelte-kit-blog-demo/create-your-blog/


### MDsveX (Svelte Markdown)

* reading time
* template, ...
* https://mdsvex.com/docs
* https://mdsvex.com/docs#integrations
* https://github.com/spences10/sveltekit-mdsvex-starter-blog/blob/main/mdsvex.config.js

### DaisyUI & Themes

* https://daisyui.com/
* https://github.com/saadeghi/daisyui
* https://github.com/saadeghi/theme-change
* https://dev.to/nico_bachner/sveltekit-theme-switch-a58


### Icons

* https://heroicons.dev/
* https://icons.getbootstrap.com/icons/
