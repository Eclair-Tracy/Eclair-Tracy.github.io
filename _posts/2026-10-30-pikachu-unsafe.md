---
title: Pikachu unsafe不安全的文件下载 Writeup
date: 2026-10-30
category: Web安全
tags: [文件下载,任意文件下载,路径穿越,目录遍历,Pikachu]
---
## 一、实验环境
靶场：Pikachu
页面：unsafedownload/down_nba.php
执行下载接口：unsafedownload/execdownload.php
漏洞类型：不安全的文件下载（路径穿越/目录遍历）

## 二、漏洞原理
页面功能：点击球员名字，调用下载接口，根据 `filename` 参数读取图片文件并提供下载。
后端直接将用户传入的 `filename` 参数拼接到文件读取函数中，**没有过滤 `../` 路径穿越符，也没有做目录限制**。
攻击者可以利用 `../` 向上跳转目录，跳出图片存放目录，读取服务器上其他任意文件。

一句话：可控文件参数 + 未过滤路径跳转符，造成任意文件读取。
底层PHP逻辑示例：
``php
$filename = $_GET['filename'];
download($filename);
## 三、测试payload

正常下载链接（业务正常功能）
http://127.0.0.1/pikachu/vul/unsafedownload/execdownload.php?filename=kb.png
基础验证Payload（读取同目录源码）
http://127.0.0.1/pikachu/vul/unsafedownload/execdownload.php?filename=../down_nba.php
读取pikachu首页源码
http://127.0.0.1/pikachu/vul/unsafedownload/execdownload.php?filename=../../index.php
拼接原理：
../ 代表向上跳转一层目录，多次使用 ../ 跳出当前图片目录，访问其他目录下的文件。
## 四、漏洞危害

1. 读取服务器敏感文件：网站源码、配置文件、数据库账号密码等

2. 源码泄露容易造成更多漏洞被挖掘

3. 配合其他漏洞，帮助攻击者进一步获取服务器权限
