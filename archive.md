---
layout: page
title: Blog Archive
lang: en
ref: archive
---

{%- assign current_lang = page.lang | default: site.lang | default: "en" -%}
{%- assign posts = site.posts | where: "lang", current_lang | sort: "date" | reverse -%}

<ul class="post-list">
{% for post in posts %}
<li>
<span class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</span>
<a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
{% if post.tags.size > 0 %}
<div class="post-tags">
{% for tag in post.tags %}
<span class="post-tag">{{ tag }}</span>
{% endfor %}
</div>
{% endif %}
</li>
{% endfor %}
</ul>
