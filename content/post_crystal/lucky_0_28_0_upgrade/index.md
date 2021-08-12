---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Lucky Framework Upgrade"
subtitle: "Upgrading from Lucky 0.27.2 to 0.28.0"
summary: "Exploring how to upgrade crystal projects (Lucky)"
authors: ["btihen"]
tags: ["upgrade", "shards"]
categories: ["Code", "Lucky", "Crystal Language"]
date: 2021-05-12T01:01:53+02:00
lastmod: 2021-08-12T01:01:53+02:00
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


## Purpose

I learned about Lucky improvements (fixes from the minor bugs after my first article) and wanted to test them out.

## Upgrading

First lets be sure we have a recent crystal version:
```bash
asdf install crystal 1.1.1
asdf global crystal 1.1.1
```

Second to be safe-side, I lets upgrade the lucky-cli:
```bash
git clone https://github.com/luckyframework/lucky_cli
cd lucky_cli
git checkout v0.28.0  # note this does not match the lucky-framework version (0.27.2)!
shards install
crystal build src/lucky.cr
cp lucky /usr/local/bin
cd ..
lucky -v  # hopefully responds with: 0.28.0
```

Now lets be sure we update `.tools-available` in the lucky project folder:

Then (in the project folder - type:
```bash
cd animals  # my lucky-project
asdf local crystal 1.1.1
```

now lets check the shards file - my current file looks like:
```ruby
# shard.yml
name: animals
version: 0.1.0

targets:
  animals:
    main: src/animals.cr

crystal: 0.36.1

dependencies:
  lucky:
    github: luckyframework/lucky
    version: ~> 0.27.0
  authentic:
    github: luckyframework/authentic
    version: ~> 0.7.3
  carbon:
    github: luckyframework/carbon
    version: ~> 0.1.4
  dotenv:
    github: gdotdesign/cr-dotenv
    version: ~> 1.0.0
  lucky_task:
    github: luckyframework/lucky_task
    version: ~> 0.1.0
  jwt:
    github: crystal-community/jwt
    version: ~> 1.5.0
development_dependencies:
  lucky_flow:
    github: luckyframework/lucky_flow
    version: ~> 0.7.3
```

So we need to update the crystal version.

Now lets updated the shards.
Go into the project folder and type:
```bash
shards update
shards list   # be sure we have lucky 0.28.0
```

hmm - lucky framework didn't upgrade to 0.28.0 -- lets try (probably shards works like ruby bundler - pinned versions need to be named in the update):
```bash
shards update lucky
shards list
```

hmm - lucky framework didn't upgrade to 0.28.0 - lets lets update the `shards` file to:
```ruby
# shard.yml
name: animals
version: 0.1.0

targets:
  animals:
    main: src/animals.cr

crystal: 1.1.1

dependencies:
  lucky:
    github: luckyframework/lucky
    version: ~> 0.28.0
  authentic:
    github: luckyframework/authentic
  carbon:
    github: luckyframework/carbon
  dotenv:
    github: gdotdesign/cr-dotenv
  lucky_task:
    github: luckyframework/lucky_task
  jwt:
    github: crystal-community/jwt
development_dependencies:
  lucky_flow:
    github: luckyframework/lucky_flow
```

OK - leys try again!
```bash
shards update lucky
shards list
```

Cool - it worked!

Now lets see if your site still works
```bash
crystal spec
```

and start up the project with:
```bash
lucky dev
```

while we are at it we should update the yarn / node packages too.
```bash
yarn upgrade
```
