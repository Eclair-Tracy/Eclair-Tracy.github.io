---
title: Pikachu SSRF实验报告
date: 2026-10-30
category: Web安全
tags: [SSRF, curl, file_get_contents, Pikachu]
---

## 一、实验环境
靶场平台：Pikachu
测试页面：
1. `vul/ssrf/ssrf_curl.php`
2. `vul/ssrf/ssrf_filegetcontent.php`
PHP版本：5.6.9
漏洞类型：SSRF（服务端请求伪造）

## 二、漏洞原理
后端接收用户可控url参数，分别使用`curl_exec()`、`file_get_contents()`访问目标资源，没有对目标地址做过滤。
SSRF的本质是**服务器代替客户端发起网络请求**，攻击者可以借助服务器的网络权限访问内网资源。
> 环境说明：Windows系统文件权限、PHP open_basedir限制，file协议读取本地文件会返回403禁止访问，因此采用HTTP内网地址验证漏洞。

## 三、测试Payload
### SSRF(curl)
1. 访问本机内网地址
http://127.0.0.1/pikachu/vul/ssrf/ssrf_curl.php?url=http://127.0.0.1
### SSRF(file_get_content)
1. 访问本机内网地址
http://127.0.0.1/pikachu/vul/ssrf/ssrf_filegetcontent.php?url=http://127.0.0.1
## 四、实验步骤
1. 访问ssrf_curl.php，传入`http://127.0.0.1`，后端curl函数替服务器访问本机，返回pikachu首页，漏洞验证成功；
2. 访问ssrf_filegetcontent.php，传入`http://127.0.0.1`，后端file_get_contents读取本机网页资源，漏洞验证成功；
3. 尝试file协议读取本地文件，受Windows权限与PHP配置限制，返回403，读取失败。

## 五、漏洞危害
1. 对内网进行端口扫描，探测内网主机与服务；
2. 攻击内网后台应用；
3. 在无权限限制的环境下，可以读取服务器本地敏感文件。

## 六、修复方案
1. 黑名单拦截127.0.0.1、内网网段；
2. 禁用file、gopher等危险协议；
3. 使用可信域名白名单；
4. 禁止直接把用户输入传入curl、file_get_contents。
