---
layout: archive
permalink: /recent-posts/
title: "Recent Posts"
---

<div class="tiles">
{% for post in site.posts %}
	{% include archive-single.html %}
{% endfor %}
</div><!-- /.tiles -->

