---
title: "effectiveJava"
layout: archive
permalink: effectiveJava
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories.effectiveJava %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}