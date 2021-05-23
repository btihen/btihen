---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Lucky Framework with Crystal Language"
subtitle: "Basics of Lucky: Relationships, Types, Forms and Language Tweeks"
summary: "A simple but reasonably comprehensive overview of Lucky features - with the context of a 'mini-project'"
authors: ["btihen"]
tags: ["Relationships", "Basics", "Forms", "Components", "Routing", "Lucky", "Web Framework", "Crystal Language"]
categories: ["Code", "Lucky", "Crystal Language"]
date: 2021-05-02T01:01:53+02:00
lastmod: 2021-05-06T01:01:53+02:00
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
https://kit.svelte.dev/
https://megzari.com/blog/about_this_site
https://svelte.dev/examples#dom-event-forwarding
https://prismic.io/blog/svelte-sveltekit-tutorial
https://www.liip.ch/en/blog/sveltekit-and-tailwind-windi-css
https://svelteland.github.io/svelte-kit-blog-demo/create-your-blog/

Darkmode Switcher: https://www.youtube.com/watch?v=o4Prej0wIZA

```
mkdir myproject
cd myproject
touch .tool-versions
echo "nodejs 14.16.0" >> .tool-versions

# creates sveltekit project
npm init svelte@next

# adds markdown capabilities
npx svelte-add mdsvex
npm install

# add tailwind via Windi/Vite
npm i -D vite-plugin-windicss

# now updates `velte.config.cjs` file:
// svelte.config.cjs
const sveltePreprocess = require('svelte-preprocess');
const node = require('@sveltejs/adapter-node');
const pkg = require('./package.json');
const WindiCSS = require('vite-plugin-windicss').default

/** @type {import('@sveltejs/kit').Config} */
module.exports = {
    // Consult https://github.com/sveltejs/svelte-preprocess
    // for more information about preprocessors
    preprocess: sveltePreprocess(),
    kit: {
        // By default, `npm run build` will create a standard Node app.
        // You can create optimized builds for different platforms by
        // specifying a different adapter
        adapter: node(),

        // hydrate the <div id="svelte"> element in src/app.html
        target: '#svelte',

        vite: {
            ssr: {
                noExternal: Object.keys(pkg.dependencies || {})
            },
            plugins: [
                WindiCSS(),
            ]
        }
    }
};

# open `src/routes/$layout.svelte` file and add this import:

<!-- src/routes/$layout.svelte -->
<script>
    import 'virtual:windi.css';
</script>

# I need to add some settings, so I use my plain old tailwind config file.
# There are two important details:
# 1. The file must be named tailwind.config.cjs (not .js)
# 2. If you need plugins, import them from windicss/plugin

# Here is my example config:

// tailwind.config.cjs
module.exports = {
    dark: 'class',
    plugins: [
        require('windicss/plugin/forms'),
        require('windicss/plugin/aspect-ratio'),
        require('windicss/plugin/line-clamp'),
        require('windicss/plugin/filters'),
        require('windicss/plugin/scroll-snap'),
    ],
}
```
