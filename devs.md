---
layout: page_minimal
permalink: /devs/
title: The Team
tagline: Minimal Mistakes, a Jekyll Theme
tags: [dev, developers, profile]
modified: 4-8-2014
comments: true
image:
  feature: texture-feature-02.jpg
  credit: Texture Lovers
  creditlink: http://texturelovers.com
---

{% for author_id in site.author_ids %}
{% assign author = site.authors[author_id] %}
  [![{{ author.name}} ]({{ author.gravatar }})]({{ author.web }})
{% endfor %}
