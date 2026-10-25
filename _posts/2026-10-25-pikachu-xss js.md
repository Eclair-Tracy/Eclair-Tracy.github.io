---
title: Pikachu - XSS之js输出 Writeup
date: 2026-10-25
categories: [Pikachu, Web安全]
tags: [XSS, JS输出注入, 反射型XSS]
---

## 靶场地址
127.0.0.1/pikachu/vul/xss/xss_04.php
## 漏洞类型
反射型XSS（JS代码内注入）

## 源码分析
php
$ms = $_GET['message'];
<script>
var ms = '$ms';
if(ms.length != 0){
    if(ms == 'tmac'){
        $('#fromjs').text('tmac确实厉害,看那小眼神..')
    }else {
        $('#fromjs').text('无论如何不要放弃心中所爱..')
    }
}
</script>
用户输入直接输出到JS变量 ms = '【输入】' 的单引号之中。没有对单引号做转义，导致JS注入。

## 利用思路

使用单引号闭合前面的 '，用分号结束原有语句，插入JS代码，最后用 // 注释掉后面多余的单引号，防止语法错误。

## Payload
';alert(1);//

## 复现步骤

1. 访问靶场页面，问题为 which NBA player do you like?

2. 输入框填入payload：';alert(1);//

3. 点击 submit 提交

4. 页面加载直接弹出 alert 弹窗，验证成功
