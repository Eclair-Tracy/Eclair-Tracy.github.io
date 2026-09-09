---
title: BugkuCTF-滑稽 Writeup
date: 2026-09-09
categories: CTF
tags: [Bugku, WEB, 前端源码]
---
# Bugku CTF 滑稽 Writeup
## 题目描述
打开页面，屏幕上充满大量滑稽表情粒子动画，右键被限制，F12被禁用。

## 解题思路
页面大量表情包是干扰，尝试查看网页源代码。
1. 使用快捷键 `Ctrl+U` 打开网页源码。
2. 在源码页面使用 `Ctrl+F`，搜索关键词`flag`。
3. 在HTML注释 `<!-- flag{xxxxxx} -->` 中找到flag。
> HTML注释里的内容浏览器不会渲染展示，但是查看源码可以直接读取，属于前端源码泄露。

## Flag
flag{9dc382fef7a0787247eee21a9d3e9f5e}

## 总结
本题考察前端基础，了解HTML注释。网页可以禁用右键、F12，但是Ctrl+U查看源码依然可以生效。
