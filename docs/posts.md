---
layout: default
title: Posts
permalink: /posts/
---

<div class="postlist">
{% for post in site.posts %}
  <div class="card">
    <div class="meta">{{ post.date | date: "%Y-%m-%d" }}</div>
    <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>
    {% if post.excerpt %}
      <p>{{ post.excerpt | strip_html | truncate: 220 }}</p>
    {% endif %}
  </div>
{% endfor %}
</div>
