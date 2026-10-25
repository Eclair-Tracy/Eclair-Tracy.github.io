---
title: Pikachu XSS-反射型XSS(POST)
date: 2026-10-25
categories: [Pikachu]
tags: [Pikachu, Web, XSS, 反射型XSS, POST]
---

## 靶场地址
`http://127.0.0.1/pikachu/vul/xss/xsspost/post_login.php`

## 漏洞简介
本关卡需要先登录，登录成功后进入输入页面。页面功能是输入喜欢的NBA球星名字，表单以POST方式提交数据。后端没有对用户输入进行过滤转义，存在反射型XSS（POST）。
和GET型反射XSS最大区别：提交的数据放在请求体中，不会显示在URL地址栏。同样属于反射型，输入内容不会存入数据库，刷新页面就消失。

## 漏洞原理
程序接收POST提交的message参数，直接拼接到HTML页面回显输出。当传入`<script>`标签的JS代码，浏览器识别脚本标签并执行里面的代码，触发XSS跨站脚本攻击。

## 复现步骤
1. 访问登录页面，账号：`admin`，密码：`123456`，点击Login登录
2. 登录成功，进入页面：`Which NBA player do you like?`
3. 在输入框填入Payload：`<script>alert(1)</script>`
4. 点击submit提交，页面弹出弹窗，漏洞验证成功

## Payload
html
<script>alert(1)</script>

## 结果
成功弹出alert弹窗，JS脚本正常执行，关卡通关。
