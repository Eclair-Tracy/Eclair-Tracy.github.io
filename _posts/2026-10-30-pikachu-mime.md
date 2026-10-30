---
title: Pikachu 文件上传-MIME type 绕过 Writeup
date: 2026-10-30
category: Web安全
tags: [文件上传,MIME校验,Content-Type,Burp,Pikachu]
---

## 一、实验环境
靶场：Pikachu
页面：unsafefileupload/mime.php
漏洞类型：不安全文件上传，MIME类型校验绕过

## 二、漏洞原理
后端通过判断上传数据包中的 `Content-Type`（MIME类型）来判断文件是否为图片，只允许 `image/jpeg`、`image/png`、`image/gif` 等图片类型。
**但是后端没有校验文件名后缀、文件真实内容**。
MIME类型是HTTP请求头，由客户端可控，攻击者抓包修改Content-Type为图片类型即可绕过校验，上传php木马。

一句话：后端仅校验请求头中的MIME类型，MIME可被抓包篡改，造成绕过。

## 三、测试payload
一句话木马内容：
``php
<?php @eval($_POST['cmd']);?>
### 利用流程：

1. 准备shell.php一句话木马，在页面选择文件上传

2. Burp抓取上传数据包

3. 将请求头里 Content-Type: application/x-php 修改为 Content-Type: image/jpeg

4. 放行数据包，上传成功，获取后门地址，蚁剑连接

## 四、漏洞危害

1. 上传PHP一句话木马，实现任意代码执行

2. 攻击者获取服务器权限，读写网站文件、植入后门

3. 长期控制服务器

## 五、修复方案

1. 不能只依靠MIME头校验，MIME头是客户端可控的，可以随意篡改

2. 后端使用后缀白名单，严格限制允许上传的后缀

3. 校验文件头部（文件头），判断文件真实内容是不是图片

4. 上传目录配置禁止执行php脚本
