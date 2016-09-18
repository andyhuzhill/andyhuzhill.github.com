---
layout: page
title : Hu Zhenyu's Blog
---
{% include JB/setup %}

# 全部文章

<ul>
{% for post in site.posts %}
<li> {{ post.date | date:"%Y-%m-%d" }}
<a href="{{ site.url }}{{ post.url }}">{{post.title}}</a>
</li>
{% endfor %}
</ul>
