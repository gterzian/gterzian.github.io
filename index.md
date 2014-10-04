---
title: Front Page
layout: front
---



Django cookbook
===============

A collaborative cookbook full of [Django](https://www.djangoproject.com/) recipes, setup as a Django project itself allowing you to download it and run the tests.
Feel free to comment and contribute!


####Signals recipes
{% for post in site.categories.signals %}

* [{{post.title}}]({{site.baseurl}}{{post.url}})

{% endfor %}

####Models recipes

{% for post in site.categories.models %}

* [{{post.title}}]({{site.baseurl}}{{post.url}})

{% endfor %}



About this repo
---------------

This repo is intended as a place where people can find and discuss recipes and patterns applicable to complex Django application that 'outgrow'
the conventions provided by the Django MVC (or [MTV](https://docs.djangoproject.com/en/dev/faq/general/#django-appears-to-be-a-mvc-framework-but-you-call-the-controller-the-view-and-the-view-the-template-how-come-you-don-t-use-the-standard-names) if you prefer).

####Problem:
Django provides a great set of tools and convention which you can completely stick to while building apps of simple to intermediate levels of complexity.
Those ensures your project is build using best practices and retains a maximum level of flexibility.

However, when you start using Django in more complex ways, the official docs aren't providing guidance on how to design your system
and make it work with the rest of Django. Usually projects start piling features unto the 'Model' parts. The M in MVC however really is only meant to handle communication with a data store. It shouldn't be used as a coathanger for all additional application logic. 
When putting everything in your models you often end up with huge model definitions and a lot of coupled and unflexible code with cannot be readily re-used. Having a model with hundreds of lines of methods also undermines Django's goal of readability, achieved through the clean declarative syntax of models fields and meta options. 

####Solution:
It is often better to separate additional features from the Django MVC and plug those into the framework using Python or Django-specific hooks. This will result in decoupled code that is re-useable, flexible and a pleasure to work with.

####Discussion:
You don't hear that much about [Design Patterns](http://en.wikipedia.org/wiki/Software_design_pattern) in Django or Python specific plublications nowadays. Perhaps those patterns
where just meant as a way to make C++ more flexible? Have they become irrelevant?

Strongly typed languages perhaps require one to think more about building flexibility into your code.
However, even in dynamic languages you need to carefully design your system or risk throwing away or rewritting your code at the smallest of changes.

Imagine you wrote a little search app as part of a Django project. Over time you are likely to find opportunities for improving your algorithms. Implementing those changes will require changing code: removing, adding or changing the return values or arguments of methods. Python gives you a lot of flexibility to make those changes while remaining backward compatible (think args and kwargs). Nevertheless, the more of your code is used directly elsewhere in the project, the more likely changing your code will result in having to rewrite those other parts as well. This is useless and painful work. Fortunately it can be prevented by clearly thinking about which parts of your app are part of it's public API and which are not and communicating this with potential users of your code. Doing so will also force you to think in terms of an API, which in itself will lead to better code. The API is nothing more then the public abstraction your code is offering to users. 

Developpers have to change code all the time. How do you ensure that change is easy, both on the author and users of the code?
Testing is one thing, it makes you quickly aware of breakages. Good design practices is another, it will prevent breakages in the first place
by promoting decoupled code. Refactoring is another important aspect, it will relief you from spending too much time with upfront design, instead spreading
the design effort over the life of the project. The goal of each is to make the inevitable change easier.

The good news is Python and Django themselves both offer you powerful tools for implementing good design
in ways which make full use of Python's [dynamic](http://en.wikipedia.org/wiki/Dynamic_programming_language) and [multi-paradigm](http://en.wikipedia.org/wiki/Programming_paradigm#Multi-paradigm_programming_language) nature. 

####Good stuff:
* [The original GOF work](http://en.wikipedia.org/wiki/Design_Patterns)
* [Refactoring Ruby edition](http://martinfowler.com/books/refactoringRubyEd.html)
* [Python 3 CookBook](http://shop.oreilly.com/product/0636920027072.do)

