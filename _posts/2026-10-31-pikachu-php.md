---
title: Pikachu PHP反序列化 Writeup
date: 2026-10-30
category: Web安全
tags: [PHP反序列化,unserialize,Pikachu]
---

## 一、实验环境
靶场：Pikachu
页面：vul/unserilization/unser.php
漏洞类型：PHP不安全反序列化

## 二、漏洞原理
页面接收用户输入的序列化字符串，直接传入`unserialize()`函数进行反序列化。
源码存在类S，反序列化后会读取`$test`变量并直接输出到页面，变量内容可控，可注入JS代码，触发XSS。
一句话：用户输入可控，直接传给unserialize()，没有校验过滤。

## 三、测试payload
O:1:"S":1:{s:4:"test";s:29:"alert('xss')";}
利用流程：
1. 进入PHP反序列化漏洞页面
2. 将payload粘贴到输入框，点击提交
3. 页面弹出alert弹窗，证明反序列化漏洞成功触发

## 四、漏洞危害
1. 如果目标类存在危险魔术方法（__destruct、__wakeup），可以实现文件读写、命令执行
2. 配合输出可实现XSS，窃取Cookie

## 五、修复方案
1. 不要反序列化来自不可信用户的输入
2. 对反序列化的对象做白名单校验，限制允许的类
3. 使用安全的格式替代PHP序列化（JSON）
