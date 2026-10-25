---
title: Pikachu XSS-存储型XSS
date: 2026-10-25
categories: [Pikachu]
tags: [Pikachu, Web, XSS, 存储型XSS]
---

## 靶场地址
`http://127.0.0.1/pikachu/vul/xss/xss_stored.php`

## 漏洞简介
这是一个留言板功能，用户输入留言提交后，内容会存入后端数据库，再展示在留言列表中。程序没有对留言内容过滤转义，存在**存储型（持久型）XSS**。
和反射型XSS不同：恶意代码保存到数据库，**所有访问该页面的用户都会触发XSS**，危害更大。

## 漏洞原理
后端接收留言，直接存入数据库；页面加载留言列表时，原样取出内容拼进HTML输出。`<script>`标签被浏览器识别并执行JS代码。

## 复现步骤
1. 访问留言板页面，在留言输入框填入payload：
html
<script>alert(1)</script>
2. 点击submit提交

3. 留言出现在留言列表，页面弹出弹窗

4. 刷新页面、新开浏览器访问本页面，依然会弹窗（证明持久存储）

## Payload
<script>alert(1)</script>

## 结果

脚本成功执行弹窗，恶意代码持久保存在数据库，存储型XSS验证成功。
