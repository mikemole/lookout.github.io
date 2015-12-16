---
layout: post
title: "Smart Seed Dump"
tags:
- jruby
- ruby
- rails
---

You've just inherited a Rails project with a complext DB schema. It has decent
unit test coverage, but you'd really like a working database so you can play
around with the running app during development. Unfortunately, there's no
seeds.rb or dev databases to play around with. What do you do? You could always
create a seeds.rb by hand, but that's going to take **forever**. You might look
around for a tool which can generate this file for you, but they all seem to
dump a random set of records from each table, preserving none of the relationships
of your data.

Enter [smart-seed-dump](https://github.com/lookout/smart-seed-dump). Smart Seed
Dump actually started as a fork of the v0.5.3 release of
[seed_dump](https://github.com/rroblak/seed_dump). It was improved upon to
leverage Rails metadata to crawl the relationship structure between models
and generate a seeds.rb file containing a logical sliver of an existing database.

Smart Seed Dump can be incorporated into your rails app by adding the
`smart-seed-dump` gem, and including the necessary railtie.
You can then produce a seeds.rb by running `rake db:seed:dump`. Various options
are also provided to tweak the output. 
See [smart-seed-dump](https://github.com/lookout/smart-seed-dump) on GitHub for details.
