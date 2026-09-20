---
title: CTFHub-HTTP-Cookie Writeup
date: 2026-09-20
category: Web
tags: [Web, Cookie, HTTP, 身份欺骗, CTFHub]
---
## 解题思路
网站直接读取客户端Cookie来判断用户身份，没有对Cookie进行后端校验，存在Cookie欺骗漏洞。
我们将Cookie中`admin`的值从0修改为1，`admin=1`代表管理员身份，伪造管理员身份发送请求，即可获取flag。
有两种实现方式：浏览器修改Cookie，或者curl命令添加请求头修改Cookie。
## 解题步骤
浏览器F12修改Cookie

1. 打开靶机页面，按下F12打开开发者工具，切换到【应用程序】标签。

2. 在左侧找到Cookie，选中靶机域名，可以看到Cookie项admin，值为0。

3. 双击admin的值，将0修改为1。

4. 刷新页面，页面输出flag。
## Flag
ctfhub{1a2732a6f1912ec92df7c751}
