---
layout: page
title: Categories
permalink: /Categories/
---

<ul>
  {% for category in site.categories %}
    <li><a href="{{ site.baseurl }}/categories/{{ category | slugify }}">{{ category }}</a> ({{ site.categories[category].size }})</li>
  {% endfor %}
</ul>
