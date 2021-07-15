---
title: "일상"
permalink: /categories/diary/
layout: category
author_profile : true
taxonomy : diary
sidebar_main: true
---

{% assign posts = site.categories.diary %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}

