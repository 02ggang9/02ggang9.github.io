---
title: "CleanCode"
layout: archive
permalink: /cleanCode
---


{% assign posts = site.categories.cleanCode %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}