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
        <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title}}</a>
    </div>
{% endfor %}
</ul>


