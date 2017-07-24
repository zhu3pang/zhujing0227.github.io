---
layout: page
title: Category
permalink: /category/
order: 4
share: false
---

<!-- <ul class="inline">
    {% for category in site.categories %}
    <li><a href="{{ '/search/?t=' | prepend: site.baseurl }}{{ category[0] }}">#{{ category[0] }}</a></li>
    {% endfor %}
</ul> -->

{% for category in site.categories %}
<h2>{{ category | first }}</h2>
<!-- </span>{{ category | last | size }}</span> -->
<ul class="arc-list">
    {% for post in category.last %}
        <li><a href="{{ post.url }}">{{ post.date | date:"%Y-%m-%d"}}&nbsp;&nbsp;{{ post.title }}</a></li>
    {% endfor %}
</ul>
---
{% endfor %}
