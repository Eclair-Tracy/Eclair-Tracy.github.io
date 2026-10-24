---
title: pikachu-暴力破解-验证码绕过(on server) Writeup
date: 2026-10-24
categories: [Pikachu]
tags: [Pikachu, 暴力破解, 验证码绕过, burpsuite, web]
---

## 靶场地址
`http://127.0.0.1/pikachu/vul/burteforce/bf_server.php`

## 漏洞简介
验证码为服务端校验，校验成功之后存入Session。在当前会话中后端仅校验一次验证码，后续请求不再重复校验验证码，攻击者可复用已经校验通过的验证码，绕过验证码防护，对密码进行暴力破解。

## 复现过程
1. Burp开启代理，使用内置浏览器访问该登录页面；
2. 填写表单：用户名admin，密码填写任意错误值，验证码填写页面展示的正确验证码，点击Login提交；
3. Proxy捕获登录POST包，右键Send to Intruder；
4. Positions中Clear §清空原有标记，仅对password参数的值添加爆破标记，vcode固定为当前有效的验证码，不添加标记；
5. Payloads选择Simple list，加载密码字典；
6. 使用Sniper模式，点击Start attack开始爆破；
7. 根据响应长度区分返回结果，爆破得到正确密码`123456`；
8. 使用账号admin，密码123456登录，登录成功。

## 请求包参考
http
POST /pikachu/vul/burteforce/bf_server.php HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=xxxxxxxxxxxx

username=admin&password=§111111§&vcode=pc1gyz&submit=Login

## 漏洞原理

后端验证码校验逻辑存在缺陷，验证码校验成功存入Session后，同一会话下不会重复校验验证码，仅校验账号密码，导致验证码防护机制被绕过。

修复方案

1. 每次登录提交请求都强制校验验证码；

2. 验证码校验一次后立即销毁Session中的验证码，缩短验证码有效期；

3. 添加登录失败次数锁定、接口限流，防范暴力破解；

4. 统一登录失败提示，减少返回信息差异。

## 小结

该关卡不是破解验证码，而是利用Session会话的校验逻辑漏洞，复用已经校验成功的验证码绕过防护。如果会话过期，验证码就会失效，需要重新提交表单获取新的有效验证码。
