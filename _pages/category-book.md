---
title: "book"
layout: archive
permalink: /book
---

{% assign posts = site.categories.book %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}%
