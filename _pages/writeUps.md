---
layout: archive
title:  "WriteUps"
permalink: /writeUps/
author_profile: true
tag: writeups
header:
  image: /assets/images/header/3.png
---


{% include group-by-array collection=site.posts field="categories" %}
<ul>
{% for cat in group_names %}
  {% if cat != "article" %}
    {% assign posts = group_items[forloop.index0] %}
    <h2 id="{{ cat | slugify }}" class="archive__subtitle">{{ cat }}</h2>
    {% for post in posts %}
      <li class="active"><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  {% endif %}
{% endfor %}
</ul>
