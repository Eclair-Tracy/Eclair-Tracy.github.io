---
title: CTFHub-RCE-读取源代码 Writeup
date: 2026-10-24
category: web
tags: [CTFHub, RCE, 文件包含, 伪协议, php://filter]
---

## 漏洞分析

### 1. 源码审计

访问靶机首页，查看页面回显：
页面提示 `flag in /flag`。
这关的考点是**利用 `php://filter` 伪协议读取文件**。因为在 `include` 时，如果直接包含 `/flag`，PHP 引擎会尝试把它当成 PHP 代码执行。如果文件内容不是合法的 PHP 代码，页面会直接报错或什么都不输出，我们就无法看到内容。

### 2. 解题思路

利用 `php://filter` 伪协议，配合 `convert.base64-encode` 过滤器。它可以在文件被 `include` 之前，先将文件内容进行 Base64 编码，这样 PHP 引擎就只会把它当成普通的字符串输出，从而完美拿到文件内容。

## 解题步骤

**构造 Payload：**
直接在浏览器地址栏访问以下 URL（不需要 POST 请求）：
http://靶机地址/?file=php://filter/read=convert.base64-encode/resource=/flag

**读取与解码：**
页面回显了一长串 Base64 字符串。
将其复制到在线 Base64 解码工具中，得到最终 Flag：
### `ctfhub{a3767c994ccc75ac2c079c30}`

