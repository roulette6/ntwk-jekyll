---
layout: page
title: Infrastructure
order: 13
---

This section covers infrastructure services, mostly in the context of Cisco IOS and NX-OS. It also tangentially covers infrastructure security in terms of some best practices when setting up network devices.

<ul>
{% assign pages = site.pages | sort: 'order' %}
{% for page in pages %}
  {% if page.category == "infrastructure" %}
<li><a href="{{ page.url }}">{{ page.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>