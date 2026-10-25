---
title: Pikachu XSS-DOM型XSS
date: 2026-10-25
categories: [Pikachu]
tags: [Pikachu, Web, XSS, DOM-XSS]
---

## 靶场地址
`127.0.0.1/pikachu/vul/xss/xss_dom.php`

## 漏洞简介
DOM型XSS，**全程在浏览器前端处理，数据不会发送给后端PHP服务器**。
页面JS获取输入框内容，拼接到 `<a href=' '>` 超链接标签中渲染，没有过滤，产生DOM XSS。

## 漏洞原理
核心代码：
js
document.getElementById("dom").innerHTML = "<a href='"+str+"'>what do you see?</a >";
用户输入的str放在href的单引号属性内，我们输入'闭合href，再追加onclick事件，实现JS执行。

## 复现步骤

1. 访问DOM型XSS页面

2. 输入框填入Payload：' onclick=alert(1)>

3. 点击 click me!，页面生成 what do you see? 超链接

4. 点击这个蓝色链接，弹出alert弹窗，验证成功

## Payload
' onclick=alert(1)>
## 结果

点击链接成功弹窗，DOM XSS漏洞验证成功。
