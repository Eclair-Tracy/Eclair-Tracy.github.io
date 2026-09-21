---
title: CTFHub-信息泄露-备份文件下载-bak文件 Writeup
date: 2026-09-21
categroy: Web
tags: [CTFHub, Web, 信息泄露, bak, 源码备份]
---
## 题目描述
访问页面提示Flag在index.php源码中。直接访问index.php只能看到渲染后的页面，无法读取PHP源码，需要找到index.php的备份文件。
## 解题步骤
1. 访问靶场页面，页面提示：`Flag in index.php source code.`
2. PHP文件直接访问会被服务器解析执行，看不到源码。尝试访问备份文件`index.php.bak`。
3. 在靶场URL后拼接 `/index.php.bak`，访问成功，下载备份文件。
4. 使用记事本打开index.php.bak，读取源码找到flag。
## Flag
ctfhub{0a8b35a54f5c58a603f5d8a7}
