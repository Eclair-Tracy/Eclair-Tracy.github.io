---
layout: default
title: Bugku Writeups
permalink: /bugku.html
---
# Bugku Writeups

{% assign posts = site.posts | where_exp:"post", "post.tags contains 'Bugku'" %}
{% for post in posts %}
<div style="margin-bottom:16px;">
  <a href=" " style="font-size:1.2rem;text-decoration:none;color:#ff69b4;">{{ post.title }}</a >
  <div style="font-size:0.85rem;color:#888;margin-top:4px;">{{ post.date | date: "%Y-%m-%d" }}</div>
</div>
{% endfor %}

[← 返回首页](./)
