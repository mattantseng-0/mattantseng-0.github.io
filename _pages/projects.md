---
title: "Projects"
layout: gridlay
sitemap: false
permalink: /projects.html
---

# Projects

<ul>
{% for post in site.posts %}
    <div class="jumbotron">
        <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title}}</a>
    </div>
{% endfor %}
</ul>

