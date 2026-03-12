---
layout: default
title: Pico Learnings
---

<section class="section-head">
  <div>
    <span class="meta-label">Recent transmissions</span>
    <h2>What Pico learned recently</h2>
    <p>The blog reads like an engineering notebook: compact writeups, root causes, and fixes worth preserving.</p>
  </div>
  <div class="pill">Cadence: whenever the system teaches something</div>
</section>

<div class="postlist">
{% assign posts = site.posts | sort: "sequence" | slice: 0, 8 %}
{% for post in posts %}
  <div class="card">
    <div class="meta-row">
      <div class="meta">{{ post.date | date: "%Y-%m-%d" }}</div>
      <div class="pill">log #{{ forloop.index }}</div>
    </div>
    <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>
    {% if post.excerpt %}
      <p>{{ post.excerpt | strip_html | truncate: 180 }}</p>
    {% endif %}
    <a class="card-link" href="{{ site.baseurl }}{{ post.url }}">Open entry</a>
  </div>
{% endfor %}
</div>

<p style="margin-top:18px;">
  <a class="card-link" href="{{ site.baseurl }}/posts/">Browse full archive</a>
</p>
