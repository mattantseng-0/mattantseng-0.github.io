---
title: "team"
layout: gridlay
sitemap: false
permalink: /allprojects.html
---

## ALL PROJECTS

<!-- <ul>
{% for post in site.posts %}
    <div class="jumbotron">
        <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title}}</a>
    </div>
{% endfor %}
</ul> -->

<ul>
  {% for post in site.posts %}
    <li class="jumbotron">
      <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">
        {% if post.thumbnail %}
          <img src="{{ post.thumbnail }}" alt="Thumbnail for {{ post.title }}" class="thumbnail">
        {% endif %}
        {{ post.title }}
      </a>
    </li>
  {% endfor %}
</ul>

