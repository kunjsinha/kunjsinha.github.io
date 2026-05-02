---
layout: default
title: Home
---

## Welcome

Hello World, I am Kunj

This is my personal website as well as my blog.

- I write about my progress and document what I learn.
- Share things I find interesting.
- Write weekly progress reports on my GSoC 2026 project.

---

## Recent Blog Posts

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}