---
title: Pikachu CSRF Token Writeup
date: 2026-10-27
category: 漏洞复现
tags: [Pikachu, Web, CSRF, 安全防御]
---

## 1. 实验环境
- 靶场：Pikachu
- 链接：`http://127.0.0.1/pikachu/vul/csrf/csrftoken/token_get_login.php`
- 工具：Chrome、F12开发者工具、Burp Suite

## 2. 实验目的
1. 理解 CSRF Token 的防御原理
2. 对比无Token的CSRF，体会Token防护效果
3. 理解同源策略和Token之间的关系

## 3. 原理简述
CSRF 跨站请求伪造：攻击者构造恶意页面，诱导已经登录的用户访问，浏览器自动携带Cookie发起请求，在用户不知情的情况下完成敏感操作。

CSRF Token 防御方式：
后端生成随机Token，绑定当前用户Session，放到表单的 `hidden` 隐藏输入框。
提交修改信息这类敏感请求时，必须带上这个Token；后端校验Token，校验不通过直接拒绝请求。
因为同源策略，跨域的恶意页面无法读取目标站点页面源码，拿不到实时Token，传统CSRF失效。

## 4. 复现步骤
1. 访问登录页面 `token_get_login.php`，使用账号 `lucy` / 密码 `123456` 登录。
2. 登录后进入会员主页 `token_get.php`，点击【修改个人信息】，跳转至 `token_get_edit.php`。
3. F12打开源码，Ctrl+F搜索token，找到隐藏表单：
``html
<input type="hidden" name="token" value="随机字符串">
4. 正常提交修改，抓包可以看到请求参数包含token。缺少token或者token过期，提交会报错 csrf token error。
5. 尝试构造传统CSRF恶意表单页面，恶意页面无法获取实时变化的token，提交请求被服务器拦截，攻击失败。

## 5. 实验结论

CSRF Token 是防御CSRF的有效方案。
它利用随机一次性token，和session绑定；跨站攻击者无法获取token，不能构造合法请求，阻止跨站伪造请求。
