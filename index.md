---
layout: default
title: 幸福的原因是元英 | 网安我来啦
---

# CTF Writeups
按平台整理题目与解题记录

<div style="display:flex;gap:20px;flex-wrap:wrap;margin:30px 0;">
{% assign bugku_posts = site.posts | where_exp:"post", "post.tags contains 'Bugku'" %}
<a href=" " data-pjax="false" style="display:block;width:320px;padding:24px;border:1px solid #ddd;border-radius:16px;text-decoration:none;color:inherit;">
  <h3 style="color:#4183c4;margin:0 0 8px 0;">BUGKU CTF</h3>
  <h2 style="margin:0 0 12px 0;">Bugku</h2>
  <p style="color:#555;">Web、Misc、Crypto、Reverse、PWN<br>等入门题目的解题记录。</p >
  <p>{{ bugku_posts.size }} 篇</p >
  <span style="color:#4183c4;">查看全部Writeup →</span>
</a >

{% assign ctfhub_posts = site.posts | where_exp:"post", "post.tags contains 'CTFHub'" %}
<a href="/ctfhub/" data-pjax="false" style="display:block;width:320px;padding:24px;border:1px solid #ddd;border-radius:16px;text-decoration:none;color:inherit;">
  <h3 style="color:#4183c4;margin:0 0 8px 0;">CTFHUB</h3>
  <h2 style="margin:0 0 12px 0;">CTFHub</h2>
  <p style="color:#555;">CTFHub 技能树与综合题目的解题记录。</p >
  <p>{{ ctfhub_posts.size }} 篇</p >
  <span style="color:#4183c4;">查看全部Writeup →</span>
</a >
</div>
