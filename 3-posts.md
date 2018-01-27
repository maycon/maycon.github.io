---
layout: page
title: Posts
permalink: /posts/
tags: main-menu
---

<ul>
	{% for post in site.posts %}
        {% if post.title %}
            <li>
            	<time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%d/%m/%Y" }}</time> &raquo;
            	<a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
           	</li>
        {% endif %}
    {% endfor %}
</ul>
