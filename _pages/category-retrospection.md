---
title: "회고"
layout: archive
permalink: /retrospection
---

{% assign posts = site.categories.retrospection %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}