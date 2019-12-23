---
layout: page
title: Routing
order: 10
---

On OSI layer 2, devices communicate with one another via their MAC address when they are in the same layer 3 network (same network ID and subnet mask). When devices in different subnets need to communicate, they require the help of routers to reach each other. This starts with their default gateway (i.e., router), which is on the same subnet and presumably knows how to get packets to the network the device is trying to reach.

For example, let's say a client wants to reach a server. The client will send a request to its default gatway destined for the server. The client's default gateway will need to know where to send the packet so it's one hop (that is, one router) closer to the server. Each successive router that gets the packet needs to make its own routing decision until the packet reaches the default gateway of the server, which then gives the packet to the server. When the server responds, the same process occurs in reverse.

This section covers how routers learn routes and how they decide which route to use if there are more than one route for a destination. 

<ul>
{% assign pages = site.pages | sort: 'order' %}
{% for page in pages %}
  {% if page.category == "routing" %}
<li><a href="{{ page.url }}">{{ page.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>