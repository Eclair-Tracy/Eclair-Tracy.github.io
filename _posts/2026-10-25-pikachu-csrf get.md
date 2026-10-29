---
title: Pikachu - CSRF(get) Writeup
date: 2026-10-25
categories: [Pikachu, Web安全]
tags: [CSRF, GET型跨站请求伪造, Pikachu]
---
## 靶场地址
127.0.0.1/pikachu/vul/csrf/csrfget/csrf_get_login.php
## 漏洞类型
跨站请求伪造 CSRF（GET型）

## 业务与原理分析
网站依靠 Cookie/Session 识别登录用户，修改个人信息接口使用 GET 请求，**没有 CSRF Token、没有额外校验**。
只要浏览器处于登录状态，访问构造好的GET链接，浏览器会自动带上Cookie，服务端就会执行修改操作。

可用账号（密码统一：`123456`）：
`vince` / `allen` / `kobe` / `kevin` / `lucy`

## 利用思路
1. 使用账号登录，维持登录Cookie
2. 正常提交修改信息，获取GET请求URL
3. 修改URL里的参数，构造恶意链接
4. 在同一个浏览器、保持登录的情况下访问恶意链接，实现信息篡改

## Payload示例（kobe）
127.0.0.1/pikachu/vul/csrf/csrfget/csrf_get_edit.php?username=kobe&sex=hacker&phonenum=666666&add=csrf&email=csrf@test.com&submit=submit
## 复现步骤
1. 访问登录页面，账号：`kobe`，密码：`123456`，点击Login登录
2. 点击【修改个人信息】，随便填写信息并提交，从地址栏拿到GET链接
3. 修改链接中的参数，构造恶意URL
4. 新建同浏览器标签页，粘贴恶意链接访问（保持登录状态）
5. 返回编辑页面刷新，个人信息被篡改，验证成功

## 漏洞成因
敏感修改接口仅依靠Cookie鉴权，缺少CSRF Token防护；操作使用GET请求，参数暴露在URL中，容易伪造链接。
