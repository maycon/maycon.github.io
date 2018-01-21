---
layout: page
title: About
permalink: /about/
---

<ul>
	{% for member in site.data.members %}
		<li>
			<a href="https://github.com/{{ member.github }}">
				{{ member.name }}
			</a>
		</li>
	{% endfor %}
</ul>
