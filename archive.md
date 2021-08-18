---
layout: page
title: Archive
---

- 2021
{% for post in site.posts %}
  {% if ((post.date | date_to_string) | slice: -4, 4) == "2021" %}
    - Some separater
    - {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% endif %}
{% endfor %}
