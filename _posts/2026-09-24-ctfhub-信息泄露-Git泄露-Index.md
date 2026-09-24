---
title: CTFHub-信息泄露-Git泄露-Index Writeup
date: 2026-09-24
category: web
tags: [CTFHub, git泄露, index泄露, 源码泄露]
---
## 题目考点

Git 源码泄露（index 索引泄露）
网站遗留 .git 目录，存在 .git/index 索引文件泄露，使用GitHacker下载git仓库，还原网站源码，找到隐藏 Flag。
## 解题过程

1. 工具准备

使用自动化 Git 泄露利用工具：GitHacker

2. 爬取完整 .git 仓库

CMD输入命令（本次解题没有使用--brute参数）：
githacker --url http://challenge-0980a8947137ce1a.sandbox.ctfhub.com:10800/.git/ --output-folder result

3. 进入真实 Git 仓库目录

GitHacker会在result文件夹内部生成一串随机长名称子文件夹，真正的git仓库放在这个子文件夹内，外层result文件夹是空的。
cd result
cd 368ae1631d5ffb1fec1f885de48cdfea

4. 查看仓库内全部文件
git ls-files
输出文件清单：

• 147631152515537.txt

• 50x.html

• index.html

5. 读取TXT文件获取 Flag

flag存放在147631152515537.txt，Windows CMD使用type命令读取文件内容：
type 147631152515537.txt
执行后输出文件内容，拿到flag。

## Flag
ctfhub{10999dc50e6d11159d163fe5}
