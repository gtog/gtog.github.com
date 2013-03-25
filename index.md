---
layout: page
title: gtog.github.com
tagline: ""
---
{% include JB/setup %}


    
## Recent Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

### Contact me:
aduncaninmotion@gmail.com