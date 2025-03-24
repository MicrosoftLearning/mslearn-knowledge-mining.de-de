---
title: Übungen zur Azure-Wissensgewinnung (Knowledge Mining)
permalink: index.html
layout: home
---

# Übungen zur Azure-Wissensgewinnung (Knowledge Mining)

Die folgenden Übungen sind eine Ergänzung für die Module in Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %} {% if activity.url contains 'ai-foundry' %} {% continue %} {% endif %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
