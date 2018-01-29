---
layout: page
title: Publicações
permalink: /br/posts/
lang: br
ref: page-posts
---

<ul>
	{% assign posts=site.posts | where:"lang", site.default_lang %}
	{% for default in posts %}
		{% assign translation = site.posts | where:"ref", default.ref | where:"lang", "en" %}
		{% if translation[0] %}
			<li>
				<time datetime="{{ translation[0].date | date:"%Y-%m-%d" }}">{{ translation[0].date | date:"%d/%m/%Y" }}</time> &raquo;
				<a href="{{ translation[0].url }}">{{ translation[0].title }}</a>
			</li>
		{% else %}
			<li>
				<time datetime="{{ default.date | date:"%Y-%m-%d" }}">{{ default.date | date:"%d/%m/%Y" }}</time> &raquo;
				<a href="{{ default.url }}">{{ default.title }}</a> [br]
			</li>
		{% endif %}
	{% endfor %}
</ul>