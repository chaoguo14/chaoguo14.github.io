---
layout: page
title: Archive
---

- 2021
{% for post in site.posts %}
  {% assign pd = (post.date | date_to_string) | slice: -4, 4 %}
  {{ pd }}
  {% if pd == "21" %}
    {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% endif %}
{% endfor %}
