---
title: "회고"
layout: archive
permalink: /retro
---

{% assign posts = site.categories.retro %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}