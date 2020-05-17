---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Github_website_with_custom_domain"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-05-17T13:22:48+02:00
lastmod: 2020-05-17T13:22:48+02:00
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

### step 1: point your domain name at: username.github.io (optional)

This is a nice step, but it takes quite a few more steps to enable https again (not covered here).

#### 1A: go to you domain dns registry

point: `domain-name.com` at `username.github.io`

I used name cheap and this article got me oriented:
https://dev.to/rightfrombasics/connecting-namecheap-domain-with-github-pages-3nn6

basically make a `CNAME` record that points `www.domain-name.com` to `username.github.io`

#### 1B: configure you github site to handle the domain

```
cd public
touch CNAME
echo 'domain-name.com' >> CNAME
git add .
git commit -m 'domain-name.com redirection'
git push
```

#### 1C: visit: `http://www.domain-name.com`

### step 2: create a valid ssl for the domain

### step 3: configure github to use https