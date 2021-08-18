---
layout: page
title: Archive
---

- 2021
{% for post in site.posts %}
  {% assign pd = (post.date | date_to_string) | slice: -4, 4 %}
  {% if pd == "2021" %}
    {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% endif %}
{% endfor %}
