---
title: Pikachu-File INclusion(remote) Writeup
date: 2026-10-30
category: web
tags:[ File Inclusion, Pikachu]
---
## 一、漏洞介绍
远程文件包含漏洞（RFI），当 PHP 配置中 `allow_url_fopen` 和 `allow_url_include` 均开启时，`include()` 等文件包含函数支持通过 http 协议加载远程文件。若 filename 参数可控且无过滤，攻击者可以构造远程文件地址，让服务器加载并执行恶意 PHP 代码。
> 注意：PHP7 及以上版本移除了 `allow_url_include`，复现需要 PHP5.6。

## 二、实验环境
- 靶场：Pikachu
- 服务：phpstudy_pro（小皮面板）
- PHP版本：php5.6.9nts
- 开启配置：`allow_url_fopen=On`、`allow_url_include=On`

## 三、复现步骤
1. 在网站根目录 `D:\phpstudy_pro\WWW` 创建 `shell.txt`，写入测试代码：
``php
<?php echo "远程文件包含测试成功！"; ?>
2. 进入 Pikachu 靶场 File Inclusion -> File Inclusion(remote)

3. 在输入框提交 payload：
http://127.0.0.1/shell.txt
4. 提交后页面输出 远程文件包含测试成功！，说明漏洞利用成功。

## 四、漏洞危害

攻击者可以加载外网恶意一句话木马，执行任意代码，控制服务器。

