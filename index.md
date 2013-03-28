---
layout: page
title: Welcome!
tagline: "-a finance and statistics blog."
---
{% include JB/setup %}

<div class="rss-chicklet">
	<p><a href="http://feeds.feedburner.com/FromGuinnessToGarch" rel="alternate" type="application/rss+xml"><img src="//feedburner.google.com/fb/images/pub/feed-icon16x16.png" alt="" style="vertical-align:middle;border:0"/></a>&nbsp;<a href="http://feeds.feedburner.com/FromGuinnessToGarch" rel="alternate" type="application/rss+xml">Subscribe to FGtoG</a></p>
</div>

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
I am currently seeking new work opportunities. Check out the [About Me](http://gtog.github.com/about_me.html) page for more details about my background.
aduncaninmotion@gmail.com
<a href="http://www.linkedin.com/in/adamcduncan">
 	<img src="http://www.linkedin.com/img/webpromo/btn_profile_bluetxt_80x15.png" width="80" height="15" border="0" alt="View Adam Duncan's profile on LinkedIn">
</a>