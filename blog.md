---
layout: page
title: Blog
permalink: /blog/
ref: blog
lang: Español
---

# Publicaciones

{% assign posts=site.posts | where:"lang", page.lang %}
{% for post in posts %}
  * ### [{{ post.title }}]({{ post.url | prepend: site.baseurl }}) <small class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</small>
    {{ post.excerpt }}
    <br>
{% endfor %}
