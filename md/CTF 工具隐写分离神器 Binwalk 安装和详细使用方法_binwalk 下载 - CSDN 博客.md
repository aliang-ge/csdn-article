> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Aluxian_/article/details/132390574?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522172009931716800222842487%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=172009931716800222842487&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-132390574-null-null.142^v100^pc_search_result_base4&utm_term=binwalk&spm=1018.2226.3001.4187)

#### [binwalk](https://so.csdn.net/so/search?q=binwalk&spm=1001.2101.3001.7020) 安装

*   *   [1.Binwalk 介绍：](#1Binwalk__2)
    *   [2.Binwalk 下载：](#2Binwalk_4)
    *   [3.Windows 安装：](#3Windows_8)
    *   [4.Linux 下载安装：](#4Linux_50)
    *   [5.Binwalk 基本用法：](#5Binwalk_57)
    *   [6.Binwalk 案例展示：](#6Binwalk_72)
    *   [7.Binwalk 总结：](#7Binwalk_81)

### 1.Binwalk 介绍：

**`Binwalk` 是用于搜索给定二进制镜像文件以获取嵌入的文件和代码的工具。 具体来说，`Binwalk`是一个固件的分析工具，旨在协助研究人员对固件非分析，提取及逆向工程用处。简单易用，完全自动化脚本，并通过自定义签名，提取规则和插件模块，还重要一点的是可以轻松地扩展。**

### 2.Binwalk 下载：

**`GitHub`项目：[https://github.com/ReFirmLabs/binwalk](https://github.com/ReFirmLabs/binwalk)**

### 3.Windows 安装：

**下载`git`项目 `cmd`运行 `python setup.py install`**

![](https://img-blog.csdnimg.cn/c6b5f097b71841a5b730ad3d3d4c294f.png)  
![](https://img-blog.csdnimg.cn/b4e15f3af7c54ce08836358176e39e82.png)

**在`python`的安装目录中的`Scripts`脚本文件夹下生成了`binwalk -h`查看 发现报错了**

![](https://img-blog.csdnimg.cn/899b0d25d0cb499383b3aed6b0a96c87.png)![](https://img-blog.csdnimg.cn/6b972132aca1494581f9df575b34a091.png)  
**这里报错是因为这个版本需要`pwd`模块 解决方法有两种：**

1.  **可以 换个版本，换个低于 `<=2.3.2` 的版本即可（我用的这种）**
2.  **安装`pwd`模块 （这个没试 大家可以试一下）**

**`binwalk2.3.2`下载：[https://github.com/ReFirmLabs/binwalk/archive/refs/tags/v2.3.2.zip](https://github.com/ReFirmLabs/binwalk/archive/refs/tags/v2.3.2.zip)**

![](https://img-blog.csdnimg.cn/e0f84493626240b8998c947e7038d7e6.png)  
![](https://img-blog.csdnimg.cn/f22ce66a909b413fafa3c7c977be918f.png)  
**🆗到这里其实已经可以正常使用了 但是为了方便点 可以写个脚本封装一下**

```
# binwalk.py
 
import os
import sys
 
file = ' '.join(sys.argv[1:])
command = "python3 D:\python\Scripts " + file
```

![](https://img-blog.csdnimg.cn/8cdde1f42e9b48db860de690a8225456.png)

**PS：这里报错了 我们安装一下即可 `pip3 install pyinstaller`**

**在执行 `pyinstaller --onefile binwalk.py`**

![](https://img-blog.csdnimg.cn/04cb15780d0a4c5691f48ab353141417.png)

![](https://img-blog.csdnimg.cn/2714ce31aa3a406c8d207f4d8e2f8abe.png)  
**然后将`binwalk.exe`复制到`C:\Windows\System32`目录下即可执行。**

### 4.Linux 下载安装：

```
git clone https://github.com/ReFirmLabs/binwalk.git
cd binwalk
python setup.py install
```

### 5.Binwalk 基本用法：

```
binwalk [选项] 文件名
```

**参数介绍：**

*   **`-B`：不执行任何提取，只显示可能包含文件的偏移量。**
*   **`-e`：将所有提取文件保存到当前目录下的一个子目录中。**
*   **`-M`：尝试包含另一个已知格式（以逗号分隔的列表）。**
*   **`-y`：尝试所有提取操作 / 文件类型。**

**PS：用的最多的就是 `binwalk -e` 分离全部到文件夹 或者`-h` 详细查看**

### 6.Binwalk 案例展示：

**这里就演示一下`CTF`题目 因为我做题的时候基本都用`kali` 这里试一下`windows`的**

![](https://img-blog.csdnimg.cn/72de73c4f5194b24ac4baf97353d9091.png)  
![](https://img-blog.csdnimg.cn/0b8ca55343f142c095d91c530ad097cc.png)  
**🆗 测试完毕 成功！！！！！！！**

**WP:`HDCTF ExtremeMisc`：[http://t.csdn.cn/qjlZ5](http://t.csdn.cn/qjlZ5)**

### 7.Binwalk 总结：

**`Binwalk`是一个功能强大的命令行工具，用于提取和分析固件文件。它可以扫描文件并从中提取有用的信息和文件，快速定位漏洞，并允许您深入了解设备的特定方面。此外，`Binwalk`不仅易于使用，而且非常灵活，并且可以与其他工具和库集成使用。使用本文中的提示和技巧，您应该能够轻松开始使用`Binwalk`，并开始探索您需要的固件，最后感觉大家的观看和支持 记得 来个三连！**