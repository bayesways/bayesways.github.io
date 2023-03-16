---
layout: page
permalink: /publications/
title: publications
description: publications by categories in reversed chronological order.
years: [2022,2021, 2018,2015, 2013]
nav: true
nav_order: 1
---


<!-- _pages/publications.md -->
For a full list refer to my [Google Scholar](https://scholar.google.com/citations?user=Po4WsHsAAAAJ&hl=en) page.

<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
