---
layout: page
title: Switching
---

This section goes over the basics of a switched network. All pages in this section are under construction. As the pages are finalized, the word "draft" will be removed from their title.

<ul>
{% assign pages = site.pages | sort: 'order' %}
{% for page in pages %}
  {% if page.category == "switching" %}
<li><a href="{{ page.url }}">{{ page.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>