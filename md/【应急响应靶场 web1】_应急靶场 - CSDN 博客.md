> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_61872115/article/details/136618102)

[文章目录](#%E6%96%87%E7%AB%A0%E7%9B%AE%E5%BD%95)

[前言](#%E5%89%8D%E8%A8%80)

[一、web1](#t0)

[1、应急响应](#t1)

[1）背景](#t2)

[2）报错处理](#t3)

[3）webshell 查杀](#t4)

[4）网站日志排查](#t5)

[5）隐藏账户](#t6)

[6）挖矿程序](#t7)

[2、渗透复现](#t8)

[1）弱口令登录](#t9)

[2）插件上传](#t10)

[3）getshell](#t11)

[总结](#t12)

前言
--

本文记录一下自己的靶场解题过程

靶场来源：知攻善防实验室

对[应急响应](https://so.csdn.net/so/search?q=%E5%BA%94%E6%80%A5%E5%93%8D%E5%BA%94&spm=1001.2101.3001.7020)内容感兴趣的可以关注一下公众号交流学习

另外对于应急响应借用小迪的话说是比较简单的内容，简单的基础是取决于你作为攻击队的水平。知道怎么攻击，那么想去排查相关信息就很容易。在平时做打靶练习时当我们完成渗透后，就可以展开对应的应急响应，能更快的巩固知识，虽然自己也很少这么做，比较尴尬，哈哈哈，共勉共勉。

一、web1
------

### 1、应急响应

#### **1）背景**

小李在值守的过程中，发现有 CPU 占用飙升，出于胆子小，就立刻将服务器关机，这是他的服务器系统，请你找出以下内容，并作为通关条件：

```
1.攻击者的shell密码
2.攻击者的IP地址
3.攻击者的隐藏账户名称
4.攻击者挖矿程序的矿池域名(仅域名)
```

相关账户密码

用户: administrator

密码: Zgsf@admin.com

#### 2）报错处理

在首次打开虚拟机时可能出现版本不兼容的问题，找到 vmx 文件修改一下版本号就行

![](https://img-blog.csdnimg.cn/direct/b944067d90b4424699a9650735640921.png)

#### 3）[webshell](https://so.csdn.net/so/search?q=webshell&spm=1001.2101.3001.7020) 查杀

打开靶机，发现桌面有 phpstudy 应用，我们开启一下

![](https://img-blog.csdnimg.cn/direct/202d153c6e5f46b98151a7f5690fd24e.png)

找到网站根目录，直接查杀一下是否存在后门文件，随便什么查杀工具都行

![](https://img-blog.csdnimg.cn/direct/dfdb7cd670294472ac3b6effa68e3942.png)

直接找到了后门程序：shell.php，上面那个是我自己上传的

打开后门文件发现后门密码

![](https://img-blog.csdnimg.cn/direct/31b7a2e7a524494ebcb7f647319d28cb.png)

搜索一下 key 值就知道密码为默认密码：rebeyond，用过都知道是冰蝎后门

#### 4）网站日志排查

然后排查一下网站的日志文件，看一下流量请求，查看一圈发现只有 apache 有日志

打开日志文件发现先是一些 get 请求，然后后续有大量的 post 请求在访问同一个地址

![](https://img-blog.csdnimg.cn/direct/fc42dd460837489a8016b8b1a194e2c3.png)

此时我们知道攻击者 IP：192.168.126.1

对于这个地址我们可以访问一下就知道，他是在暴力破解账户密码

![](https://img-blog.csdnimg.cn/direct/b638cfe409714092ac4e0b7e8aeee8fa.png)

查看后续日志可发现它破解成功了，很可能是存在[弱口令](https://so.csdn.net/so/search?q=%E5%BC%B1%E5%8F%A3%E4%BB%A4&spm=1001.2101.3001.7020)

![](https://img-blog.csdnimg.cn/direct/26b43a3f620440a28a32d9719534dc7d.png)

并且在后台访问 / content/plugins/tips / 目录，成功将 shell.php 后门传上去了

![](https://img-blog.csdnimg.cn/direct/2517cd0625374278b78818cfa7f5cb00.png)

后面存在大量的连接后门的日志，那就是攻击者取得 shell 后的操作了

#### 5）隐藏账户

题目要求找到攻击者的隐藏账户，可以直接到控制面板查看或者注册表中

![](https://img-blog.csdnimg.cn/direct/98fbd9309353404da79888635651551c.png)

![](https://img-blog.csdnimg.cn/direct/769e5dc3e5dd4cf8a555667cfe1a7777.png)

找到隐藏账户 hack168

还可以从登录日志上查看有无登陆记录，直接查看事件查看器中的安全日志筛选 4624 审核成功日志记录也很多

可借助日志分析工具，公众号也可获取，当然也有很多其他的日志分析工具

查看到远程桌面登陆成功日志，发现新用户：hack168

![](https://img-blog.csdnimg.cn/direct/f7f8fbb0b0f4447cbbf5f8c94164277e.png)

#### 6）挖矿程序

先在寻找挖矿程序，并找到它的外联域名信息

找到用户文件夹下的 hack168 下的桌面文件，发现程序信息

![](https://img-blog.csdnimg.cn/direct/bf86b45d83e145e186bf273362281697.png)

后续涉及一些反编译内容，这部分我就不太了解了，搜索一下也能反编译出来，感兴趣的可以去看公众号文章。

列举用到的工具：

pyinstxtractor 反编译工具：[GitHub - extremecoders-re/pyinstxtractor: PyInstaller Extractor](https://github.com/extremecoders-re/pyinstxtractor "GitHub - extremecoders-re/pyinstxtractor: PyInstaller Extractor")

pyc 反编译工具：[pyc 反编译 - 工具匠](https://toolkk.com/tools/pyc-decomplie "pyc反编译 - 工具匠")

最终得到矿池域名：wakuang.zhigongshanfang.top

![](https://img-blog.csdnimg.cn/direct/b322542e026440f0baaebeed28e7a351.png)

这里有个很坑的地方，本来我是想偷懒不去反编译的，直接输答案算了，结果每次输完域名窗口就关闭了，试了很多次，我以为我答案错了，但是不可能啊。

ok, 我猜估计是没有暂停，我不反编译挖矿程序了，我反编译解题程序

![](https://img-blog.csdnimg.cn/direct/7f81550dbc934955952ada68e37ffe2c.png)

加了延时就正常了（这里就是先用 pyinstxtractor-master 工具反编译 exe 文件，再找到 pyc 文件使用在线 pyc 反编译工具得到源码，进行查看）

### 2、渗透复现

#### 1）弱口令登录

来到网站可以看到是 emlog cms, 一搜就能搜出历史漏洞，我们参照攻击者的思路去入侵

![](https://img-blog.csdnimg.cn/direct/9caaf9be33014604b9ee410195f6dcf9.png)

首先是来到了登录界面，采用了爆破的方式得到了账户密码

![](https://img-blog.csdnimg.cn/direct/a92f56cbcf3c470a9f1347e5a205e54b.png)

一测试，账号密码：admin/123456

#### 2）插件上传

登陆后看到了版本信息：pro 2.2.0

![](https://img-blog.csdnimg.cn/direct/b6f991c10013400fbe2e5cf369c532fd.png)

搜索发现有个插件上传漏洞，细节就不说了，可以搜索相关文章

这里要上传 zip 文件，需要注意的是压缩包文件中必须有一个与文件夹同名的 php 文件，比如文件夹名是 abc，那么你文件夹下面就需要有一个 abc.php，并且 abc.php 中需要有内容，才能正常安装

我这里是 shell.zip，里面有个 shell 的文件夹，下面有个 shell.php 的脚本

![](https://img-blog.csdnimg.cn/direct/f24eeeafd1fa4a7fb461bfdbaabb91d2.png)

可以看到上传成功

![](https://img-blog.csdnimg.cn/direct/45edfa4db7f146e7938ad4dc9595d46b.png)

这里有个坑是要关闭掉服务器的病毒与威胁防护，否则会杀掉文件。

当然作者出题的时候也是关闭了的，不然上传的后门也会被杀，这就体现服务器安全基线配置的重要性了。

#### 3）getshell

使用蚁剑连接后门

![](https://img-blog.csdnimg.cn/direct/9c684c5366fd483b943cfe990b77cff5.png)

成功获取 shell，并且为管理员权限

![](https://img-blog.csdnimg.cn/direct/c7f514b0575348dd855233f05c2abe2f.png)

总结
--

一边渗透一边应急，Fighting！