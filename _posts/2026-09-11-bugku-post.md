title: BugkuCTF-POST Writeup
date: 2026-09-11
categories: 【CTF，Web】
tags: [Web, PHP, POST传参]
---

Bugku CTF - POST

问题描述

访问题目页面，页面展示核心PHP源码：
$what=$_POST['what'];
echo $what;
if($what=='flag'){
    echo 'flag{xxx}';
}
题目需要满足参数判断条件，才可输出对应flag。

解题思路

1. 代码中使用 $_POST['what'] 接收数据，只识别POST请求体参数，URL的GET传参无法生效。

2. GET参数存放于URL地址栏，POST参数存放于数据包Body请求体中。

3. 我们需要手动构造POST表单请求，传入参数 what=flag，满足if判断条件即可获取flag。

解题步骤

1. 打开靶场题目页面，按下键盘 F12 打开浏览器开发者工具，切换到网络(Network)面板，勾选保留日志，刷新页面抓取数据包。

2. 右键抓取到的页面数据包，选择编辑并重新发送。

3. 将原本的 GET 请求方法 修改为 POST。

4. 切换到 Body 表单栏目，选择 x-www-form-urlencoded 表单格式。

5. 新增表单参数：参数名 what，参数值 flag。

6. 点击发送请求，在响应预览界面即可查看返回的flag。

旗帜

flag{09aa4e40059b3fb656a216d8f2db6229}

总结知识点

1. GET传参：参数携带在URL地址栏中，由 $_GET 接收。

2. POST传参：参数携带在请求体Body内，由 $_POST 接收。

3. PHP超全局变量区分严格，GET、POST参数不能通用，需根据代码逻辑构造对应请求方式。
