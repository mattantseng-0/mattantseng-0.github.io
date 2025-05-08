---
title: "Publications"
layout: gridlay
sitemap: false
permalink: /publications/
years: [2016, 2017, 2018, 2019, 2020, 2021]
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

<style>
.thumbnail {
  max-width: 10px; /* Adjust size as needed */
  height: auto;
  border-radius: 5px; /* Optional: add rounded corners */
}
</style>




<!-- <div class="jumbotron">
### Preprints
{% bibliography --query @unpublished %}
</div> -->

<div class="jumbotron">
### Published articles
{% bibliography --query @article %}
</div>

<div class="jumbotron">
### Master's Thesis
{% bibliography --query @thesis %}
</div>


<!-- <div class="jumbotron">
### Refereed conference proceedings
{% bibliography --query @inproceedings %}
</div> -->
