---
title: CTFHub-HTTP-请求方法Writeup
date: 2026-09-19
catecory: CTFHub
tags: [CTFHub, Web]
---

# HTTP Method

## 题目描述
了解HTTP请求方法，拿到flag。

## 解题思路
这道题考察 HTTP 请求方法，网页提示需要使用指定请求方法访问页面，不能用默认GET。

## 解题过程
1. 打开题目页面，观察页面提示，提示需要使用特定请求方法。
2. 使用工具发送对应Method请求，这里可以用curl

打开Windows cmd
输入curl.exe -X CTFHUB http://challenge-438c238715de39fa.sandbox.ctfhub.com:10800/index.php
回车键得出flag

## Flag
ctfhub{92c144cbfae01acf4cde022c}
