---
layout: default
title: Technical Terms
---

# The Friendly Dictionary of Technical Terms

{% assign sorted_terms = site.terms | sort: 'title' %}
<ul>
{% for term in sorted_terms %}
  <li><a href="{{ term.url }}">{{ term.title }}</a></li>
{% endfor %}
</ul>
