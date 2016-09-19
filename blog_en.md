---
layout: page
title: Blog
permalink: /blog-en/
ref: blog
lang: english
---

# Posts

{% assign posts=site.posts | where:"lang", page.lang %}
{% for post in posts %}
  * ### [{{ post.title }}]({{ post.url | prepend: site.baseurl }}) <small class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</small>
    {{ post.excerpt }}
{% endfor %}