---
title: "Projects"
layout: gridlay
sitemap: false
permalink: /projects.html
---

<style>
.jumbotron{
    padding:3%;
    padding-bottom:10px;
    padding-top:10px;
    margin-top:10px;
    margin-bottom:30px;
}
</style>

<style>
.thumbnail {
  max-width: 100px; /* Adjust size as needed */
  height: auto;
  border-radius: 5px; /* Optional: add rounded corners */
}
</style>
<style>
.post-link {
  display: flex;
  justify-content: space-between;
  align-items: center;
  text-decoration: none;
  color: inherit;
  width: 100%;
}
</style>

<style>
.post-title {
  flex: 1;
  font-size: 1.5em;
  padding-right: 1em; /* Add space between title and thumbnail */
}
</style>


# Projects

<!-- <ul>
  {% for post in site.posts %}
    <li class="jumbotron">
      <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">
        {{ post.title }}
        {% if post.thumbnail %}
          <img src="{{ post.thumbnail }}" alt="Thumbnail for {{ post.title }}" class="thumbnail">
        {% endif %}
      </a>
    </li>
  {% endfor %}
</ul> -->

<ul>
  {% for post in site.posts %}
    <li class="jumbotron">
      <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}" class="post-link">
        <div class="post-title">{{ post.title }}</div>
        {% if post.thumbnail %}
          <img src="{{ post.thumbnail }}" alt="Thumbnail for {{ post.title }}" class="thumbnail">
        {% endif %}
      </a>
    </li>
  {% endfor %}
</ul>