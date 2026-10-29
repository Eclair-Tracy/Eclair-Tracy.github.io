---
title: Pikachu - XSS之htmlspecialchars
date: 2026-10-25
categories: [Pikachu, Web安全]
tags: [XSS, htmlspecialchars绕过, 反射型XSS, Pikachu]
---

## 靶场地址
http://127.0.0.1/pikachu/vul/xss/xss_02.php

## 漏洞类型
反射型XSS

## 源码分析
php
$message=htmlspecialchars($_GET['message']);
echo "<a href=' '>你的输入已经被记录:{$message}</a >";
1. 使用 htmlspecialchars() 过滤，没有加 ENT_QUOTES

◦ 默认转义：< > & "

◦ 不会转义英文单引号 '

2. 输入同时输出在两个位置：

◦ href='{$message}' 属性里面

◦ 超链接展示文字：>{$message}</a >

## 利用思路

href 属性支持 javascript: 伪协议，点击链接就执行JS代码。不需要尖括号，不受< >转义影响，是本关卡主流解法。

## Payload
javascript:alert(1)

## 复现步骤

1. 访问靶场页面

2. 输入框填入 payload：javascript:alert(1)

3. 点击 submit 提交

4. 页面生成蓝色超链接，点击链接，弹出 alert 弹窗，验证成功
