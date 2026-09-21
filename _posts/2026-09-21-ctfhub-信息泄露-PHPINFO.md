---
title: CTFHub-信息泄露-PHPINFO Writeup
date: 2026-09-21
categroy: Web
tags: [CTFHub, Web, 信息泄露， phpinfo]
---
## 题目描述
访问靶场链接，页面为phpinfo信息页面，需要在页面内找到flag。
## 解题思路
phpinfo()函数会输出服务器操作系统、PHP版本、环境变量等大量敏感信息。页面没有访问限制，造成信息泄露。flag保存在页面的环境变量中，使用浏览器查找功能快速定位。
## 解题步骤
1. 打开靶场链接，访问 phpinfo.php
2. 页面展示PHP全部配置信息，内容很长。
3. 使用浏览器快捷键 `Ctrl + F`，打开页面搜索功能，搜索关键词`flag`。
4. 页面自动跳转到Environment（环境变量）区域，找到FLAG的值，复制flag。
## Flag
ctfhub{759d600bb172714468bb0263}
