---
title: BugkuCTF-source Writeup
date: 2026-09-11
categories: [CTF, Web]
tags:[Web, git泄露, 源码泄露, 目录扫描]
---

## 题目描述

访问目标网站存在git源码泄露，通过git-dumper下载.git仓库，利用git reflog查找历史提交记录，找到被删除的flag

## 解题步骤

下载git仓库源码
在cmd执行命令http://160.202.254.160:16606/.git/ ./source_git
下载完成后，目录生成source_git文件夹
进入仓库目录
cd source_git
查看git所有历史提交记录git reflog

结果
d256328 HEAD@{0}: reset: moving to d25632
13ce8d0 HEAD@{1}: commit: flag is here?
fdce35e HEAD@{2}: reset: moving to fdce35e
e0b8e8e HEAD@{3}: reset: moving to e0b8e
40c6d51 HEAD@{4}: commit: flag is here?
fdce35e HEAD@{5}: commit: flag is here?
d256328 HEAD@{6}: master
e0b8e8e HEAD@{7}: commit (initial): this is index.html
记录commit哈希
查看历史提交记录获取内容

## Flag

flag{git_is_good_distributed_version_control_system}
