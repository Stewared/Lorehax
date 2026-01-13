---
layout: post
title: Home
---

# Recent Posts

{% for post in site.posts %}
  <article class="post-preview">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p class="post-date"><small>{{ post.date | date: "%B %d, %Y" }}</small></p>
    {% if post.thumbnail %}
      <a href="{{ post.url | relative_url }}">
        <img src="{{ post.thumbnail | relative_url }}" alt="{{ post.title }}" class="post-thumbnail">
      </a>
    {% endif %}
  </article>
{% endfor %}
