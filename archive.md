---
layout: page
title: Archive
---

{% assign all_years = "2021,2020,2019" | split: "," %}
{% for y in all_years %}
- {{ y }}
{% for post in site.posts %}
  {% assign pd = (post.date | date_to_string) | slice: -4, 4 %}
  {% if pd == y %}
  - {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% endif %}
{% endfor %}
{% endfor %}
