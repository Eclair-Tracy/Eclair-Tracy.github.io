---
layout: default
title: 幸福的原因是元英 | 网安我来啦
---

<style>
body {
  background-color: #ffffff !important;
  color: #222222 !important;
}

.page-title {
  text-align: center;
  padding: 40px 20px 10px;
  font-size: 32px;
  font-weight: bold;
  color: #222222 !important;
}

.page-desc {
  text-align: center;
  color: #666666 !important;
  font-size: 16px;
  margin-bottom: 30px;
}

.card-wrap {
  max-width: 720px;
  margin: 0 auto;
  padding: 0 20px 40px;
}

.ctf-card {
  background-color: #f5f7fa !important;
  border: 1px solid #e3e7ee !important;
  border-radius: 20px;
  padding: 28px;
  margin-bottom: 20px;
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.04);
}

.card-tag {
  color: #2f6feb !important;
  font-size: 15px;
  font-weight: 600;
}

.card-title {
  font-size: 34px;
  font-weight: bold;
  margin: 6px 0 10px;
  color: #222222 !important;
}

.card-desc {
  color: #444444 !important;
  font-size: 16px;
  line-height: 1.7;
}

.card-count {
  margin-top: 16px;
  font-size: 15px;
  color: #555555 !important;
}

.card-link {
  display: inline-block;
  margin-top: 14px;
  color: #2f6feb !important;
  text-decoration: none;
  font-weight: 500;
}

.card-link:hover {
  text-decoration: underline;
}
</style>

<div class="page-title">CTF Writeups</div>
<div class="page-desc">按平台整理题目与解题记录</div>

<div class="card-wrap">
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
