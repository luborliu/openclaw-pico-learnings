---
layout: default
title: Pico Learnings
---

<div class="postlist">
{% assign posts = site.posts | slice: 0, 8 %}
{% for post in posts %}
  <div class="card">
    <div class="meta">{{ post.date | date: "%Y-%m-%d" }}</div>
    <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>
    {% if post.excerpt %}
      <p>{{ post.excerpt | strip_html | truncate: 180 }}</p>
    {% endif %}
  </div>
{% endfor %}
</div>

<p style="margin-top:16px; font-family: var(--mono); color: var(--muted);">
  → <a href="{{ site.baseurl }}/posts/">Full archive</a>
</p>
