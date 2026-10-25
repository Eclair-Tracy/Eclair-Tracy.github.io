---
title: Pikachu XSS-XSS盲打
date: 2026-10-25
categories: [Pikachu]
tags: [Pikachu, Web, XSS, XSS盲打]
---

## 靶场地址
前台：`http://127.0.0.1/pikachu/vul/xss/xssblind/xss_blind.php`
后台：`http://127.0.0.1/pikachu/vul/xss/xssblind/admin_login.php`

## 漏洞简介
XSS盲打：用户输入内容提交后**不在前端页面回显**，存入数据库；只有管理员登录后台查看留言数据时，内容才会被渲染，触发XSS。
提交之后前台没有任何弹窗反馈，必须切换到管理员后台验证。

## 漏洞原理
后端接收留言，没有过滤，存入数据库；管理员后台页面取出留言直接拼接HTML渲染，JS代码执行。

## 复现步骤
1. 打开盲打前台页面，看法输入框填入payload：`<script>alert(1)</script>`
2. 大名随便填写，点击提交，页面提示感谢参与，前台无弹窗
3. 访问后台登录页面，账号admin，密码123456登录后台
4. 进入留言列表页面，自动弹出alert弹窗，漏洞验证成功

## Payload
html
<script>alert(1)</script>
## 结果

管理员后台加载留言时触发弹窗，XSS盲打成功。
