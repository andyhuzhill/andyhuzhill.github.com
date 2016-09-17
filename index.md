---
layout: page
title : 所有文章
---
{% include JB/setup %}

<ul>
{% for post in site.posts %}
<li> {{ post.date | date:"%Y-%m-%d" }}
<a href="{{ site.url }}{{ post.url }}">{{post.title}}</a>
</li>
{% endfor %}
</ul>
