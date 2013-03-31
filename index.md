---
layout: page
title: Welcome!
tagline: "-a finance and statistics blog."
---
{% include JB/setup %}

<div class="rss-chicklet">
	<p><a href="http://feeds.feedburner.com/FromGuinnessToGarch" rel="alternate" type="application/rss+xml"><img src="//feedburner.google.com/fb/images/pub/feed-icon16x16.png" alt="" style="vertical-align:middle;border:0"/></a>&nbsp;<a href="http://feeds.feedburner.com/FromGuinnessToGarch" rel="alternate" type="application/rss+xml">Subscribe to FGtoG</a></p>
</div>

This blog is a place where I post articles I have written about topics in finance, 
data visualization, R, hacking, workflow, and other data and information projects.
I hope you find the posts useful in your own work. Feel free to access any of the 
files in my github repositories. This site is hosted on github using Jekyll. There
are a number of useful guides on how to create your own blog and host it on 
Github using Jekyll. I will write up my own experiences getting this site up and 
running in some future posts. 

Enjoy!
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
I am currently seeking new work opportunities. Check out the [About Me](http://gtog.github.com/about_me.html) page for more details about my background or head over to my profile on Linked In. 
<a href="http://www.linkedin.com/in/adamcduncan">
 	<img src="http://www.linkedin.com/img/webpromo/btn_profile_bluetxt_80x15.png" width="80" height="15" border="0" alt="View Adam Duncan's profile on LinkedIn">
 </a>.
aduncaninmotion@gmail.com