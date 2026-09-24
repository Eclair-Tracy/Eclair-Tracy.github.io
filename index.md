---
layout: default
title: 幸福的原因是元英 | I AM
---

<style>
  body {
    background-image: url("/yes.jpg");
    background-repeat: no-repeat;
    background-size: cover;
    background-attachment: fixed;
  }
</style>

<span style="color:#ff69b4;font-size:1.1rem;">
  🎀 <a href=" " style="color:#ff69b4;text-decoration:none;">关于我</a > 🎀
</span>

# <span style="color:#ff69b4;">CTF Writeups</span>
<span style="color:#666666;">按平台整理题目与解题记录</span>

{% assign bugku_posts = site.posts | where_exp:"post", "post.tags contains 'Bugku'" %}
## <span style="color:#ff69b4;">BUGKU CTF</span>
<span style="color:#666666;">Bugku Web、Misc、Crypto、Reverse、Pwn等入门题目的解题记录。<br>共 {{ bugku_posts.size }} 篇</span>

<span style="color:#666666;">[查看全部writeup ->](./bugku.html)</span>

{% assign ctfhub_posts = site.posts | where_exp:"post", "post.tags contains 'CTFHub'" %}
## <span style="color:#ff69b4;">CTFHub</span>
<span style="color:#666666;">CTFHub技能树与综合题目的解题记录。<br>共 {{ ctfhub_posts.size }} 篇</span>

<span style="color:#666666;">[查看全部writeup ->](./ctfhub.html)</span>
