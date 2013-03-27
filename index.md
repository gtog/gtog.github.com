---
layout: page
title: Welcome!
tagline: "-a finance and statistics blog."
---
{% include JB/setup %}

    
## Recent Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


### R-Bloggers.com
Please check out [R-bloggers.com](http://www.r-bloggers.com) for more awesome posts about all things R.


### Contact me:
Thanks for reading! Feel free to contact me with comments. 
Oh yeah, after 17 years, I'm unemployed and available for hire. Check out the [About Me](http://gtog.github.com/about_me.html) page for more details about my background.
aduncaninmotion@gmail.com