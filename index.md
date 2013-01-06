---
layout: page
title: Recent Posts
tagline: Wyatt's JavaScript && Node Blog
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li>
      <strong><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></strong>
      <div>{{ post.description }}</div>
    </li>
  {% endfor %}
</ul>
