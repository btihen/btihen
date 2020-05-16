---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Publish Hugo Website using Github"
subtitle: "Using the Academic Theme"
summary: "githug and submodules to publish a hugo site"
authors: ["Bill Tihen"]
tags: ["Hugo", "Static Site", "git", "submodules"]
categories: ["code"]
date: 2020-05-16T10:39:21+02:00
lastmod: 2020-05-16T10:39:21+02:00
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

## install hugo

```
brew install hugo
```

## create an `username_website` repo on your github

I'll assume your github account is `username`

## clone the academic repo locally

```
git clone https://github.com/sourcethemes/academic-kickstart.git username_website
cd academic_website
git submodule update --init --recursive  # without this the site won't start correctly
```

be sure you have many files within: **`themes/academic`**

## point this repo to your `username_website` repo

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[remote "origin"]
	url = git://github.com/sourcethemes/academic-kickstart.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```

change the origin url usuing:
```
sed -i.bak -e 's/https:\/\/github.com\/sourcethemes\/academic-kickstart.git/git@github.com:username\/username_website.git/' .git/config
```

Now this file should look like:
```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[remote "origin"]
	url = git@github.com:username/username_website.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```

now you can push this local repo to your github repo using:

```
git push -u origin --all
# git init
# git add .
# git commit -m "Initial commit"
# git push -u origin master
```

## configure website basics

### Set your site name:

in `config/_default/config.toml`

find the `title` attribute and set it to `username` (or whatever is appropriate)

### Pick a themes

from https://sourcethemes.com/academic/themes/

in `config/_default/config.toml`

find the `theme` attribute and set it to your favorite theme color (or leave it as is)

### site logo & favicon

Put your image files into assets/images:
* `logo.png` (the logo on your webpage) file and
* `icon.png` (the favicon - icon in the webtab)

You can go to `https://www.namecheap.com/logo-maker` and make a logo

### menu items

in `config/_default/menus.toml`

remove any items you won't use.  In my case this file now looks like:

```
[[main]]
  name = "Posts"
  url = "#posts"
  weight = 20

[[main]]
  name = "About"
  url = "#about"
  weight = 50

[[main]]
  name = "Contact"
  url = "#contact"
  weight = 60
```

These will also be the sections on the home page that will be enabled and configured.

The larger the weight the further to the **right** the item will be shown.

## configure site parameters

You may want to read through all the params - but the ones listed here are enough to get started.

* **site_type** -- in the file: `config/_default/params.toml`: be sure to configure the `site_type` variable
* **configure 'contact details'** 
  - if you choose not to add an email, then be sure to set the variable `email_form=0` on the `content/home/contact.md` file!
  - if you choose not to enter an address and coordinates the in the `[map]` section set the `engine=0` to avoid problems.
* **configure social details** -- optional
* **Regional Settings** -- NOTE: The date display settings seems to have a bug -- so I don't recommend adjusting that.

## configure your homepage

At this point I suggest starting `hugo server` so you can watch your edits.

Now go into the folder `content/home` and we will adjust or disable the files in this folder.
* disable with: `active=false`
* enable with: `active=true`
* oder with: `weight=20` the bigger the number the further down on the page is show (I suggest you use the same weights used in the menu)

* **`contact.md`** - review and see if changes are desired.
* **`accomplishments.md`** - (and all other home page sections you decide not to display) change `active=true` to **`active=false`**

### `about` page

* **`about.md`** - I have changed the title to `about` -- NOTE: the material displayed here comes from the page: `content/authours/admin` (or whatever name you change the folder to -- be sure to use the same name in `content/home/about.md` in the variable `author = "admin"`)

Now adjust the file: `content/authors/admin/_index.md` -- below the `---` toward the end of the file you can add your own free text to the about page.

now add a nice image (of the person or org to this folder and call it: **`avatar.jpeg`**)

### `people` (or Team) page

To create a list of peolple and their bios for a site.

edit **`people.md`**
- set `active=true`
- create reasonable group name(s) in the section (for example):
```
[content]
  # Choose which groups/teams of users to display.
  #   Edit `user_groups` in each user's profile to add them to one or more of these groups.
  user_groups = ["Educators", "Agilists"]
```

Now create additional folders in: `content/authors/`
for example I might make a page for me with:
```
mkdir content/authors/btihen
cp content/authors/admin/_index.md content/authors/btihen/_index.md
```

now edit: `content/authors/btihen/_index.md`

it is important to add the variable: `user_groups = ["Educators"]` (one or more of the groups)

of course edit the settings to match the person's profile.

### when the site is good enough in your localbrowser its time to take a git snapshot and publish.

edit your `.gitignore` file to track `public` (just delete this line)

now create your git snapshot:
```
git add .
git commit -m "First draft of homepage"
git push
```

now we will generate the `static` site for our webpage
```
hugo -d public
git add .
git commit -m "added static pages to publish"
```

## setup your site to to publish a github webpage!

This is a little tricky, but quite workable.

First we need to make a second repo on github called: `username.github.io` it MUST be exactly your username.github.io for this to work.

CREATE with A README (or add one) before now -- before this next step!

now go back into your website code and type:
```
git clone https://github.com/username/username.github.io.git public
```
if you see: `warning: You appear to have cloned an empty repository.` -- you will have problems!

Now add the username.github.io repo as a submodule to your website code repo using:
```
git submodule add -b master https://github.com/username/username.github.io.git public`
```

now in `.git/config` you might see:
```
[submodule "themes/academic"]
  path = themes/academic
  url = https://github.com/gcushen/hugo-academic.git

[submodule "public"]
  path = public
  url = https://github.com/username/username.github.io.git
  branch = master
```

if not add it with:
```
cat <<"EOF" >> git/config
[submodule "public"]
  path = public
  url = https://github.com/username/username.github.io.git
  branch = master
EOF
```

NOW go to github repo **settings** and click on **manage access** and be sure you have permission to at administer (or at least write to this repo) -- probably not so click the **`invite teams or people`** button and add yourself as an admin (an other as needed).


## publish your new github webpage:

do the following:
```
hugo -d public
cd public
git add .
git commit -m "first webpage content"
git push
# toward the end you should see: `To github.com:username/username.github.io.git`
cd ..
```

if you get the error:
```
remote: Permission to peakchallenges/peakchallenges.github.io.git denied to btihen.
fatal: unable to access 'https://github.com/peakchallenges/peakchallenges.github.io.git/': The requested URL returned error: 403
```

then go back and fix the permissions in the last step.

If that still is a problem, then re-create your local repo `username.github.io.git` outside the webcode project.
```
git clone git@github.com:username/username.github.io.git
cd username.github.io
# update the readme
git commit -am "update readme and test permissions"
git push
```
assuming this works:
```
rm -rf username_website/public
mv username.github.io username_website/public
cd username_website/public
# update the readme again
git commit -am "update readme and test permissions within webcode"
git push
```

assuming this now works do:
```
cd ..
hugo -d public
cd public
git add .
git commit -m "publish new content on: xx-xx-xxxx"
git push
cd ..
git status
# commit if needed
```

now to go https://username.github.io/ (it might take a few minutes to publish)
