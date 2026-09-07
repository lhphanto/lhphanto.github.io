---
title: "Building XLeRobot"
permalink: /xlerobot/
layout: single
author_profile: true
classes: wide
---

Notes from building [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) — an
open-source dual-arm mobile robot — and what I learn along the way.

Posts are listed oldest first, so this reads as a build log from the start.

{% assign series = site.categories.xlerobot | sort: "date" %}
{% if series.size > 0 %}
  {% for post in series %}
<h2 class="archive__item-title" style="margin-bottom:0.25em">
  <span style="opacity:0.5">{{ forloop.index }}.</span>
  <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
</h2>
<p class="page__meta" style="margin-top:0">
  <i class="far fa-calendar-alt" aria-hidden="true"></i>
  {{ post.date | date: "%B %-d, %Y" }}
  {% assign wpm = site.words_per_minute | default: 200 %}
  · {{ post.content | number_of_words | divided_by: wpm | plus: 1 }} min read
</p>
{% if post.excerpt %}<p>{{ post.excerpt | markdownify | strip_html | strip | truncate: 220 }}</p>{% endif %}
<hr>
  {% endfor %}
{% else %}
*No posts yet.*
{% endif %}
