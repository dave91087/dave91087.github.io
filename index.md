---
layout: page
title: Home
tagline: Developing in the Free World
---
{% include JB/setup %}

## Recent Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> <br>
        {{ post.excerpt | remove: '<p>' | remove: '</p>' }}
    </li>
  {% endfor %}
</ul>

<br><br>

