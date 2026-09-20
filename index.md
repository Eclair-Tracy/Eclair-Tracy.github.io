---
layout: default
title: 幸福的原因是元英 | I AM
---

<span style="color:#ff69b4;">[👋 关于我](./about)</span>

# <span style="color:#ff69b4;">CTF Writeups</span>
<span style="color:#666666;">按平台整理题目与解题记录</span>

{% assign bugku_posts = site.posts | where_exp:"post", "post.tags contains 'Bugku'" %}

## <span style="color:#ff69b4;">BUGKU CTF</span>
<span style="color:#666666;">Bugku<br>Web、Misc、Crypto、Reverse、PWN<br>等入门题目的解题记录。<br>共 {{ bugku_posts.size }} 篇</span>

<span style="color:#666666;">[查看全部Writeup →](./bugku.html)</span>

{% assign ct fhub_posts = site.posts | where_exp:"post", "post.tags contains 'CTFHub'" %}

## <span style="color:#ff69b4;">CTFHUB</span>
<span style="color:#666666;">CTFHub<br>CTFHub技能树与综合题目的解题记录。<br>共 {{ ctfhub_posts.size }} 篇</span>

<span style="color:#666666;">[查看全部Writeup →](./ctfhub.html)</span>
