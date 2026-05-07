---
layout: default
title: Cool Stuff
---

## These are the things I find cool

Basically the things that I find interesting, inspiring or worth revisiting.  <br>

<ul>
{% for item in site.cool %}
  <li>
    <a href="{{ item.url | relative_url }}">
      {{ item.title }}
    </a>
  </li>
{% endfor %}
</ul>

