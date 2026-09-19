---
layout: default
title: 幸福的原因是元英 | 网安我来啦
---

<style>
.page-title{
  text-align:center;
  padding:30px 0 10px;
  font-size:42px;
  font-weight:bold;
  color:#222;
}
.page-desc{
  text-align:center;
  color:#555;
  font-size:18px;
  margin-bottom:40px;
}
.card-wrap{
  max-width:700px;
  margin:0 auto;
  padding:0 20px;
}
.ctf-card{
  background-color:#f7f8fa;
  border-radius:24px;
  padding:32px;
  margin-bottom:24px;
  border:1px solid #e5e7eb;
}
.card-tag{
  color:#346edb;
  font-size:20px;
}
.card-title{
  font-size:48px;
  font-weight:bold;
  margin:8px 0 12px;
  color:#222;
}
.card-desc{
  color:#444;
  font-size:17px;
  line-height:1.6;
}
.card-count{
  margin-top:20px;
  font-size:18px;
  color:#555;
}
.card-link{
  display:inline-block;
  margin-top:16px;
  color:#346edb;
  text-decoration:none;
}
.card-link:hover{
  text-decoration:underline;
}
</style>

<div class="page-title">CTF Writeups</div>
<div class="page-desc">按平台整理题目与解题记录</div>

<div class="card-wrap">
  <!-- BugkuCTF 卡片 -->
  <div class="ctf-card">
    <div class="card-tag">BUGKU CTF</div>
    <div class="card-title">Bugku</div>
    <div class="card-desc">
      Web、Misc、Crypto、Reverse、PWN<br>
      等入门题目的解题记录。
    </div>
    {% assign bugku_posts = site.posts | where_exp:"post", "post.tags contains 'Bugku'" %}
    <div class="card-count">{{ bugku_posts.size }} 篇</div>
    <a class="card-link" href=" ">查看全部Writeup →</a >
  </div>

  <!-- CTFHub 卡片 -->
  <div class="ctf-card">
    <div class="card-tag">CTFHUB</div>
    <div class="card-title">CTFHub</div>
    <div class="card-desc">
      CTFHub 技能树与综合题目的解题记录。
    </div>
    {% assign ctfhub_posts = site.posts | where_exp:"post", "post.tags contains 'CTFHub'" %}
    <div class="card-count">{{ ctfhub_posts.size }} 篇</div>
    <a class="card-link" href="/ctfhub.html">查看全部Writeup →</a >
  </div>
</div>
