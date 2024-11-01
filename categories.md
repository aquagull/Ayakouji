---
layout: page
title: Categories
permalink: /Categories/
---

<ul>
  {% for category in site.categories %}
    <li>
      <a href="{{ site.baseurl }}/categories/{{ category[0] | slugify }}">{{ category[0] }}</a> ({{ category[1].size }})
    </li>
  {% endfor %}
</ul>
