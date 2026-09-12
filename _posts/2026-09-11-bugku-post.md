Bugku POST writeup

题目

访问页面，页面显示PHP代码：
$what=$_POST['what'];
echo $what;
if($what=='flag'){
    echo 'flag{xxx}';
}
分析

$_POST['what'] 代表接收POST请求中，参数名为what的数据。
这道题不能用GET传参，GET参数放在URL，$_POST读取不到。我们需要构造POST请求，在请求体传入 what=flag，当变量$what等于字符串flag时，页面输出flag。

解题步骤（浏览器F12抓包）

1. 打开靶场页面，按下F12打开开发者工具，切换到【网络】面板，勾选Keep log，刷新页面，抓到当前页面的GET数据包。

2. 右键抓到的这条请求，选择编辑并重新发送。

3. 在请求编辑界面，把请求方法从GET修改为POST。

4. 切换到Body标签，选择x-www-form-urlencoded表单格式。

5. 添加表单参数：

◦ key（参数名）：what

◦ value（参数值）：flag

6. 点击蓝色Send发送请求。

7. 在右侧预览/响应区域，页面返回内容中就出现flag。

Flag

flag{09aa4e40059b3fb656a216d8f2db6229}

总结知识点

1. GET的参数放在URL；POST参数放在数据包的请求体Body里面。

2. PHP中$_POST['参数名']专门读取POST表单提交的数据，GET传参无法触发判断。...
