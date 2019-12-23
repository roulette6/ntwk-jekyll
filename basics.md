---
layout: page
title: Basics
order: 1
---

This section goes over the basics of networking so the routing and switching sections will make sense. Starting with the OSI model is a good first step before learning about Ethernet (OSI layer 2) and IP (OSI layer 3).

<ul>
{% assign pages = site.pages | sort: 'order' %}
{% for page in pages %}
  {% if page.category == "basics" %}
<li><a href="{{ page.url }}">{{ page.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>