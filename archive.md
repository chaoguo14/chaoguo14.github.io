---
layout: page
title: Archive
---

- 2021
{% for post in site.posts %}
  {% if ((post.date | date_to_string) | slice: -4, 4) == "2021" %}
    - {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% else %}
    - No idea
  {% endif %}
{% endfor %}

 - 2020
{% for post in site.posts %}
  {% if ((post.date | date_to_string) | slice: -4, 4) == "2020" %}
    - {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% endif %}
{% endfor %}

 - 2019
{% for post in site.posts %}
  {% if ((post.date | date_to_string) | slice: -4, 4) == "2019" %}
    - {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
  {% endif %}
{% endfor %}
