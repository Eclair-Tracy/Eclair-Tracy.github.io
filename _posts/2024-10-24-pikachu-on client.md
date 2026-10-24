---
title: Pikachu 暴力破解 - 验证码绕过(on client)
date: 2026-10-24
categories: [Pikachu]
tags: [Pikachu, 暴力破解, 验证码绕过, burpsuite, web]
---

## 靶场地址
`http://127.0.0.1/pikachu/vul/burteforce/bf_client.php`

## 漏洞简介
验证码校验逻辑写在前端JS中，**后端不对验证码做校验**。
错误验证码只会在浏览器弹窗拦截，不会发请求；只要抓到一次合法提交包，后端只校验账号密码，验证码字段可以固定甚至直接删掉，实现绕过验证码暴力破解。

## 复现过程
1. 打开靶场页面，打开Burp代理、开启Intercept拦截；
2. 填入账号admin、任意密码、页面上显示的正确验证码，点击Login抓包；
3. 将抓到的POST包发送到 Intruder；
4. 攻击类型选择**狙击手（Sniper）**，仅对password参数添加payload标记；
5. 加载密码字典，开始爆破；
6. 根据响应长度区分结果，找到登录成功的密码 `123456`；
> 补充：发包时vcode验证码参数可以固定不变，也可以直接删掉该参数，后端都不会校验验证码。

## 请求包参考
http
POST /pikachu/vul/burteforce/bf_client.php HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded

username=admin&password=123456&vcode=58Y05&login=Login

## 漏洞原理

验证码生成与判断全部由前端JavaScript实现，后端服务端没有接收并校验验证码的逻辑。前端JS只是单纯的客户端限制，攻击者抓包绕过前端JS校验后，可直接对账号密码爆破。
