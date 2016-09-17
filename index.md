---
layout: page
title : 
---
{% include JB/setup %}

# 文章列表

<ul>
{% for post in site.posts %}
<li> {{ post.date | date:"%Y-%m-%d" }}
<a href="{{ site.url }}{{ post.url }}">{{post.title}}</a>
</li>
{% endfor %}
</ul>
