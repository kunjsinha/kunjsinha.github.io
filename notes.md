---
layout: default
title: Field Notes
---

## The things that I find interesting, inspiring or worth revisiting.  <br>

<ul>
{% for item in site.notes %}
  <li>
    <a href="{{ item.url | relative_url }}">
      {{ item.title }}
    </a>
  </li>
{% endfor %}
</ul>

