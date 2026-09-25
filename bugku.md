---
layout: default
title: Bugku Writeups
permalink: /bugku.html
---

<style>
body {
background-image: url("/u.jpg");
background-repeat: no-repeat;
background-size: cover;
background-attachment: fixed;
}
</style>

# <span style="color:#b04874;">Bugku Writeups</span>
<span style="color:#594338;">Bugku平台题目记录</span>

{% for post in site.posts %}
{% if post.tags contains "Bugku" %}
<div style="margin:12px 0;">
<a href="_posts" style="color:#b04874;text-decoration:none;font-weight:bold;display:block;">{{post.title}}</a>
<p style="color:#594338;margin:4px 0;">{{post.date | date: "%Y-%m-%d"}}</p>
</div>
{% endif %}
{% endfor %}

<br>
<a href="./" style="color:#000000;text-decoration:none;">← 返回首页</a>
