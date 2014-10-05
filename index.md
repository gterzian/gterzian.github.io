---
title: Front Page
layout: front
---


Hello World!
===============

####It's a beautiful day in [Lujiazui](http://en.wikipedia.org/wiki/Lujiazui)

![My helpful screenshot](http://res.cloudinary.com/djfwqxjdx/image/upload/v1412525982/IMG_4221_za4ccb.jpg)




{% for post in site.posts %}

 <li><a href='{{site.baseurl}}{{post.url}}'>{{post.title}}</a></li>

{% endfor %}


