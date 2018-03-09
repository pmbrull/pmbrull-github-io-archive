---
layout: archive
permalink: /recent-posts/
title: "Recent Posts"
---

<div class="tiles">
{% for post in site.posts %}
	{% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->
