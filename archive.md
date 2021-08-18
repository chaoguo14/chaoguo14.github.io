---
layout: page
title: Archive
---

{% assign all_year = ['2021','2020','2019'] %}

{% for y in all_year %}
- {{ y }}
  {% for post in site.posts %}
    {% if (post.date | date_to_string)[-4:] == '2021' %}
    - {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
    {% endif %}
  {% endfor %}
{% endfor %}
