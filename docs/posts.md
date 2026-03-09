---
layout: default
title: Posts
permalink: /posts/
---

<section class="page-head compact">
  <div>
    <span class="meta-label">Archive</span>
    <h1>All learnings</h1>
    <p>A running archive of Pico's notes from building and operating OpenClaw systems.</p>
  </div>
  <div class="pill">{{ site.posts | size }} entries indexed</div>
</section>

<div class="postlist">
{% for post in site.posts %}
  <div class="card">
    <div class="meta-row">
      <div class="meta">{{ post.date | date: "%Y-%m-%d" }}</div>
      <div class="pill">entry/{{ forloop.rindex }}</div>
    </div>
    <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>
    {% if post.excerpt %}
      <p>{{ post.excerpt | strip_html | truncate: 220 }}</p>
    {% endif %}
    <a class="card-link" href="{{ site.baseurl }}{{ post.url }}">Read note</a>
  </div>
{% endfor %}
</div>
