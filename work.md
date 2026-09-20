---
layout: page
title: Work
permalink: /work/
wide: true
---
Projects I want people to find without digging through repositories. Newest first.

<div class="card-grid">
  {% assign projects = site.projects | sort: "date" | reverse %}
  {% for project in projects %}
    {% include project-card.html %}
  {% endfor %}
</div>
