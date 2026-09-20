---
title: CTFHub-HTTP-响应包源代码Writeup
date: 2026-09-20
category: Web安全
tags: [Web, HTTP, 响应源码, 信息收集, CTFHub]
---

## 题目信息
靶场地址：`http://challenge-f43b9ff783793381.sandbox.ctfhub.com:10800`
考点：HTTP响应包源代码查看，前端源码信息收集

## 题目描述
访问靶场页面，展示贪吃蛇小游戏。游戏结束弹窗提示“游戏结束”，通关游戏无法获取flag。flag直接写在网页的响应源代码中。

## 原理
浏览器访问网页时，服务器把HTML、JS等源码作为HTTP响应包返回给浏览器渲染页面。前端页面的静态资源（HTML、JS）中经常会直接存放注释信息、flag等敏感内容。

## 解题步骤
1. 打开靶场链接，页面加载贪吃蛇小游戏。
2. 使用快捷键 `Ctrl + U` 查看网页源代码。
3. 在源码中搜索关键词 `ctfhub{`，找到注释内的flag。
4. 提交flag完成解题。

## Flag
ctfhub{e0df5f1883d7ed0450f0d110}
