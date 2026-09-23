---
layout: default
title: 幸福的原因是元英 | I AM
---

<span style="color:#ff69b4;">[👉 关于我](./about/)</span>

# <span style="color:#ff69b4;">CTF Writeups</span>
<span style="color:#666666;">按平台整理题目与解题记录</span>

{% assign bugku_posts = site.posts | where_exp:"post", "post.tags contains 'Bugku'" %}
## <span style="color:#ff69b4;">BUGKU CTF</span>
<span style="color:#666666;">Bugku Web、Misc、Crypto、Reverse、Pwn等入门题目的解题记录。<br>共 {{ bugku_posts.size }} 篇</span>

<div style="margin-top:12px;">
{% for post in bugku_posts %}
<div style="margin-bottom:16px;">
  <a href=" " style="font-size:1.2rem;text-decoration:none;color:#ff69b4;">{{ post.title }}</a >
  <!-- 日期，直接显示在标题下方，灰色小字 -->
  <div style="font-size:0.85rem;color:#888;margin-top:4px;">{{ post.date | date: "%Y-%m-%d" }}</div>
</div>
{% endfor %}
</div>

<span style="color:#666666;">[查看全部writeup ->](./bugku.html)</span>

{% assign ctfhub_posts = site.posts | where_exp:"post", "post.tags contains 'CTFHub'" %}
## <span style="color:#ff69b4;">CTFHub</span>
<span style="color:#666666;">CTFHub技能树与综合题目的解题记录。<br>共 {{ ctfhub_posts.size }} 篇</span>

<div style="margin-top:12px;">
{% for post in ctfhub_posts %}
<div style="margin-bottom:16px;">
  <a href="{{ post.url }}" style="font-size:1.2rem;text-decoration:none;color:#ff69b4;">{{ post.title }}</a >
  <!-- 日期，直接显示在标题下方，灰色小字 -->
  <div style="font-size:0.85rem;color:#888;margin-top:4px;">{{ post.date | date: "%Y-%m-%d" }}</div>
</div>
{% endfor %}
</div>

<span style="color:#666666;">[查看全部writeup ->](./ctfhub.html)</span>
