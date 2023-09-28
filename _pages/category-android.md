---
title: "android"
layout: archive
permalink: /android
---


{% assign posts = site.categories.android %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}