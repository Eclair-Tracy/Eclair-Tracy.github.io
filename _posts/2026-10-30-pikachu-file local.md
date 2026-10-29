---
title: File Inclusion(local) 本地文件包含 Writeup
date: 2026-10-30
category: [web]
tags: [file inclusion, Pikachu]
---
## 1. 实验环境

靶场：Pikachu漏洞练习平台
访问地址：127.0.0.1/pikachu/vul/fileinclude/fi_local.php
漏洞类型：本地文件包含漏洞（LFI）

## 2. 漏洞原理

后端直接将GET传入的filename参数放到include()函数加载文件，没有过滤../目录穿越符。
简化源码：
include($_GET['filename']);
攻击者可以使用../向上跳转目录，读取服务器本地任意文件。

## 3. 实验步骤

1. 正常页面：下拉框选择球员，URL为?filename=file1.php，加载对应的球员介绍页面。

2. 构造路径穿越Payload读取系统hosts文件：
?filename=../../../../Windows/System32/drivers/etc/hosts
3. 访问该URL，页面成功输出Windows hosts文件内容，包含IP与域名映射记录，证明目录穿越+本地文件包含生效。

4. 拓展Payload（php伪协议读取PHP源码）
?filename=php://filter/read=convert.base64-encode/resource=fi_local.php
## 4. 漏洞危害

1. 读取服务器上任意本地敏感文件，获取配置、域名映射、账号信息；

2. 结合文件上传漏洞，包含图片马，解析执行PHP代码，获取Webshell，控制服务器。

