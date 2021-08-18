---
layout: page
title: Archive
---

 - 2021
   {% for post in site.posts %}
       {% if (post.date | date_to_string)[-4:] == '2021' %}
   - {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
      {% endif %}
        {% endfor %}
 - 2020
 - 2019
 
