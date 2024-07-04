> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_52395989/article/details/132189964)

以 buuctf 中的题目《附加[字符串](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1001.2101.3001.7020)：图种》为例：  
题目链接：https://buuoj.cn/challenges#[%E7%AC%AC%E4%B8%83%E7%AB%A0][7.2.1%20%E9%99%84%E5%8A%A0%E5%AD%97%E7%AC%A6%E4%B8%B2]%E5%9B%BE%E7%A7%8D

### 查看图片属性

一般较简单的[图片隐写](https://so.csdn.net/so/search?q=%E5%9B%BE%E7%89%87%E9%9A%90%E5%86%99&spm=1001.2101.3001.7020)可能在属性这里就直接给出了 flag，本题没有。  
![](https://img-blog.csdnimg.cn/db4c458c8dcf44ce821526821c09377d.png)

### 使用 010Editor 打开图片

这里你也可以使用 [WinHex](https://so.csdn.net/so/search?q=WinHex&spm=1001.2101.3001.7020)。![](https://img-blog.csdnimg.cn/a04d71aea18847189bf61fbdcfcfdaa1.png)  
这里发现它存在一个 flag.txt 文档，那我们就可以想到文件分离。

zip 文件的结尾以一串 504B0506 开始，即图片中可能包含一个压缩包压缩的 flag 文件。

### 使用 binwalk 进行文件分离。

tips: 这里需要把图片改成. jpg 的格式，之前. png 试了好几次分离都不成功。  
1. 把文件直接拖入 kali 的桌面，在搜索栏搜索 binwalk（工具是 kali 自带的，无需安装）  
![](https://img-blog.csdnimg.cn/b27888698b594c8fb8989133c22886e3.png)

### 打开 binwalk,cd 进入桌面

![](https://img-blog.csdnimg.cn/f550ab292f6040389f05ca635e5356b1.png)  
![](https://img-blog.csdnimg.cn/22dd70d97f2a4b71a7ee724d40b916e3.png)

### 查看并分离文件

**查看文件**：binwalk 文件名  
通过 binwalk 我们可以看到这一张 jpg 文件中藏着 zip 文件。  
**分离文件**：binwalk 文件名 -e  
![](https://img-blog.csdnimg.cn/73e1a03edc4c4d5d87efc3e305dcd87e.png)  
"-e" 和–extract" 用于按照定义的配置文件中的提取方法从固件中提取探测到的文件系统。

### 查看分离文件

若提取成功则会生成一个_文件名_extracted 的目录，目录中存放的就是提取出的文件，打开可以看到包含 flag 的文件。  
![](https://img-blog.csdnimg.cn/ee6cadbac46b4313a3d2f21b8837573a.png)  
![](https://img-blog.csdnimg.cn/5f23b68096ba4f4c82371e2953992255.png)