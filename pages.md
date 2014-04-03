---
layout: layout
title: Pages
subtitle: all pages of documents
---
<h2 id="Pages">All Pages</h2>
<ul>
{% for page in site.pages %}
<li><a href="{{ page.url }}">{{ page.title }}</a></li>
{% endfor %}
</ul>
