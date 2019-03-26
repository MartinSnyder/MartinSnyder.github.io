---
layout: post
title:  Site Updates
---

When I first moved this site to Jekyll/GitHub Pages almost 3 years ago, I took the "quick and dirty" approach and used permalinks on every page to try to preserve as much of the structure as possible.

I've learned a lot about Jekyll since then, and recently completed an effort to rebuild the site to take better advantage of Jekyll and kramdown functionality. This means that I broke a bunch of links, and for that I apoligize. I updated my [404 page][404] to offset most of the damage.
<!--break-->

I offer the following advice for people new to Jekyll:
* Try to avoid redundant front matter as they inflict unnecessary maintenance on the site
* When you are linking to internal site resources, use the {{ "{% link ... " }}%} syntax. Jekyll will validate those links when you generate the static site.
* RTFM - GitHub pages documentation is good, but if you want to learn Jekyll that's a bad place to start. Go straight to the [Jekyll docs][jekyll-docs] instead.

[jekyll-docs]: https://jekyllrb.com/docs/
[404]: {% link 404.md %}
