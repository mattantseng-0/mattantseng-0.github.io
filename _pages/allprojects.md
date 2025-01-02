---
title: "team"
layout: gridlay
sitemap: false
permalink: /allprojects.html
---

## ALL PROJECTS



<ul>
{% for post in site.posts %}
    <div class="jumbotron">
        <li>{{ post.date | date_to_string }}: <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title}}</a></li>
    </div>
{% endfor %}
</ul>


