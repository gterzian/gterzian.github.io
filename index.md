---
title: Front Page
layout: front
---



Hello World!
===============

[My helpful screenshot]({{ site.url }}/assets/IMG_4221.jpg)



####Signals recipes
{% for post in site.categories.signals %}

* [{{post.title}}]({{site.baseurl}}{{post.url}})

{% endfor %}

####Models recipes

{% for post in site.categories.models %}

* [{{post.title}}]({{site.baseurl}}{{post.url}})

{% endfor %}


