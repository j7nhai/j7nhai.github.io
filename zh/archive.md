---
layout: page
title: 文章归档
lang: zh
ref: archive
---

{%- assign current_lang = page.lang | default: site.lang | default: "en" -%}

{% for tag in site.tags %}
{%- assign posts = tag[1] | where: "lang", current_lang -%}
{%- if posts and posts.size > 0 -%}
<h3>{{ tag[0] }}</h3>
<ul>
{% for post in posts %}
<li><a href="{{ post.url | relative_url }}">{{ post.date | date: "%Y-%m" }} - {{ post.title }}</a></li>
{% endfor %}
</ul>
{%- endif -%}
{% endfor %}
