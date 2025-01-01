---
title: "Projects"
layout: textlay
sitemap: false
permalink: /allprojects.html
---

## Projects

<div class="jumbotron">
{% for project in site.data.projects %}
<b>{{ project.date }}</b>

{{ project.title }}
{% endfor %}

</div>
