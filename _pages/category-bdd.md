---
title: "bdd"
layout: archive
permalink: /bdd
---

{% assign posts = site.categories.bdd %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}