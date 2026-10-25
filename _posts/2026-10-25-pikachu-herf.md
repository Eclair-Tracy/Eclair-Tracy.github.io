---
title: Pikachu - XSS之href输出
date: 2026-10-25
categories: [Pikachu, Web安全]
tags: [XSS, href伪协议, 反射型XSS]
---

## 靶场地址
http://127.0.0.1/pikachu/vul/xss/xss_03.php

## 漏洞类型
反射型XSS

## 源码分析
php
$url = $_GET['url'];
echo "<a href=' '>{$url}</a >";

用户输入的内容，直接放到a标签的href属性中，同时作为链接文本展示。
本关卡没有过滤javascript伪协议。

## 利用思路

a标签的href支持javascript:伪协议，点击链接的时候，冒号后面的js代码直接执行。

## Payload
javascript:alert(1)

## 复现步骤

1. 访问靶场页面

2. 在输入框填入payload：javascript:alert(1)

3. 点击submit提交

4. 页面生成蓝色超链接，点击链接，弹出alert弹窗，验证成功
