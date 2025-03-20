---
title: Latihan Penggalian Pengetahuan Azure
permalink: index.html
layout: home
---

# Latihan Penggalian Pengetahuan Azure

Latihan berikut dirancang untuk mendukung modul di Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %} {% if activity.url contains 'ai-foundry' %} {% continue %} {% endif %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
