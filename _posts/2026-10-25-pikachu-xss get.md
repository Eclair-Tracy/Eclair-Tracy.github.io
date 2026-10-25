---
title: Pikachu XSS-反射型XSS(GET)
date: 2026-10-25
categories: [Pikachu]
tags: [Pikachu, Web, XSS, 反射型XSS]
---

## 靶场地址
`http://127.0.0.1/pikachu/vul/xss/xss_reflected_get.php`

## 漏洞简介
页面功能：询问用户喜欢的NBA球星，接收输入并回显内容。
程序没有对用户输入的特殊符号做过滤与转义，存在**反射型XSS（GET）**。
输入内容放在URL的GET参数里，服务器接收后直接反射回页面，**不会存入数据库，刷新页面就消失**。

## 漏洞原理
后端直接把 `message` 参数的内容拼到HTML页面输出。
当传入 `<script>` 标签的JS代码时，浏览器识别脚本标签并执行代码，触发XSS。
页面输入框存在前端长度限制 `maxlength="20"`，该限制仅前端可见，属于客户端限制，后端没有校验，可以通过修改URL直接绕过。

## 复现步骤
1. 访问靶场页面，提示：`Which NBA player do you like?`
2. 随便输入内容提交，观察地址栏，参数通过GET传递：
?message=kobe&submit=submit
3. 修改URL里 `message` 后面的值，替换为payload：
?message=alert(1)&submit=submit
4. 回车访问，弹出alert弹窗，攻击成功。

> 补充：也可以F12选中输入框，删除 `maxlength="20"` 后直接在输入框提交payload。本次直接修改URL绕过，没有手动修改前端属性。

## Payload
html
<script>alert(1)</script>
## 结果

页面成功弹出弹窗，JS脚本执行，漏洞验证成功。
