---
layout: default
title:  Udemy Ramda Course
---

# {{ page.title }}

{% for post in site.posts %}
  [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

<small style="position:fixed;bottom:0;left:.25rem;">
  [&copy; bmordan](https://github.com/bmordan)
</small>
