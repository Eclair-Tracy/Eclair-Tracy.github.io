---
title：BugkuCTF-LinuxWriteup
date：2026.09.16
category：CTF
tag：Misc
---
## 题目描述

下载附件压缩包，通过Linux命令查找文件中的flag。

## 解题思路

本题考察Linux基础解压命令与二进制文件读取。文件为嵌套压缩包，Windows直接解压会出现权限异常，使用WSL依次解压，通过strings命令提取二进制文件中的可读字符串，筛选出flag。

## 解题过程

1. 进入Windows桌面目录，查看题目附件
cd /mnt/c/Users/25964/Desktop
ls
成功查看到文件 linux .zip。

2. 安装解压工具并解压zip文件
sudo apt install unzip
unzip "linux .zip"
解压得到linux文件夹，包含嵌套压缩包。

3. 进入linux目录
cd linux
ls
发现已自动解压出test文件夹。

4. 进入test文件夹，查看目标文件
cd test
ls
得到二进制文件flag，无法直接读取。

5. 安装工具，提取并筛选flag
sudo apt install binutils
strings flag | grep key
成功读取到flag。

## Flag
key{feb81d3834e2423c9903f4755464060b}
## 知识点

1. WSL通过 /mnt/c/ 访问Windows磁盘文件

2. 含空格文件名需要双引号包裹解压

3. strings命令可读取二进制文件可读字符串

4. 多层压缩包的Linux解压处理方法
