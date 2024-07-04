> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/ttovlove_/article/details/135070900?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522172009931716800222842487%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=172009931716800222842487&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-6-135070900-null-null.142^v100^pc_search_result_base4&utm_term=binwalk&spm=1018.2226.3001.4187)

#### 一、引言

1. 介绍 [binwalk](https://so.csdn.net/so/search?q=binwalk&spm=1001.2101.3001.7020) 的作用和应用场景

   - 作为一款开源的二进制分析工具，binwalk 可以帮助用户提取、识别和分析各种文件中的隐藏信息，如[文件类型](https://so.csdn.net/so/search?q=%E6%96%87%E4%BB%B6%E7%B1%BB%E5%9E%8B&spm=1001.2101.3001.7020)、字符串、关键字等。

2. 为什么选择 binwalk 作为二进制分析工具

   - binwalk 具有强大的功能和灵活的参数设置，适用于多种操作系统和文件类型，且易于学习和使用。

#### 二、安装 binwalk

1. 系统要求

   - Windows、Linux 和 macOS 系统均可使用。

2. 下载 binwalk 安装包

   - 访问 binwalk 官方网站（https://github.com/ReFirmLabs/binwalk）下载最新版本的安装包。

**kali 中使用 apt 命令来安装**

```
     - -e：指定文件扩展名，用于过滤分析结果。
 
     - -M：指定最小文件大小，用于过滤分析结果。
 
     - -A：指定文件类型，用于过滤分析结果。
```

3. 安装 binwalk

   - 对于 Windows 用户，运行安装包并按照提示进行操作；对于 Linux 和 macOS 用户，使用包管理器（如 apt、yum 或 brew）进行安装。

4. 验证安装是否成功

   - 打开命令行或终端，输入 “binwalk --version” 并按回车键，若显示 binwalk 的版本信息，则表示安装成功。

```
binwalk --version
```

#### 三、基本使用方法

1. 获取待分析文件

   - 从网络下载、从设备中提取或从其他来源获取待分析的二进制文件。

2. 使用 binwalk 分析文件

   - 在命令行或终端中输入 “binwalk 文件路径” 并按回车键，binwalk 将自动开始分析文件。

   - 常用选项说明：   

```
     - -e：指定文件扩展名，用于过滤分析结果。
 
     - -M：指定最小文件大小，用于过滤分析结果。
 
     - -A：指定文件类型，用于过滤分析结果。
```

3. 查看分析结果

   - 提取文件信息：使用

```
--dd
```

或

```
--disasm
```

选项查看文件的详细信息和汇编代码。

   - 识别文件类型：使用

```
--file-type
```

选项查看文件的类型和格式。

   - 查找特定[字符串](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1001.2101.3001.7020)或关键字：使用

```
--string
```

或

```
--keyword
```

选项搜索文件中的特定内容。

   - 提取隐藏信息（如密码、密钥等）：使用

```
--entropy
```

或

```
--crc
```

选项检查文件中的加密和完整性信息。

#### 四、高级功能与技巧

1. 使用脚本自动化分析

   - 编写自定义脚本，实现批量分析和处理二进制文件的功能。

2. 与其他工具结合使用（如 Wireshark、IDA Pro 等）

   - 将 binwalk 的分析结果导入到其他工具中，进行进一步的处理和分析。

3. 自定义输出格式和报告生成

   - 使用 “--output” 选项指定输出格式，如 CSV、JSON 等；使用 “--report” 选项生成详细的分析报告。

4. 常见问题与解决方法

   - 解决分析过程中遇到的各种问题，如错误提示、分析失败等。

#### 五、实例演示

1. 分析一个已知的恶意软件样本

   - 使用 binwalk 提取恶意软件中的隐藏信息，如加密算法、通信协议等。

2. 分析一个加密的文件或数据包

   - 使用 binwalk 识别加密算法和密钥，辅助解密和破解工作。

3. 分析一个包含隐藏信息的二进制文件

   - 使用 binwalk 查找文件中的敏感信息，如用户名、密码等。

###### **以下是分离图片中隐藏文件的一个简单实例：**

**题目：png 食用**

![](https://img-blog.csdnimg.cn/direct/8e4d5879275940a0ba7cfad519eaab63.png)

用 010 以及 stegsolve 查看过图片，没有异常，怀疑是图片中隐藏文件，使用 binwalk 分析

命令如下：

```
binwalk 图片路径
```

![](https://img-blog.csdnimg.cn/direct/23f3862878c14f218a25ad4db584edcd.png)

通过 binwalk 分析可以发现在 **70658 字节**有一个 zip 的文件，将它分离出来，使用 --dd 命令：

```
dd if ="文件路径"  of="输出文件" skip=从哪个字节开始 bs=1
```

![](https://img-blog.csdnimg.cn/direct/4ed126d274994e87a9919c77edeb7b52.png)

在文件中找到分离出来的文件解压，得到隐藏内容。

（文章简单且基本，旨在教学简单使用 binwalk 分离文件，后续会逐渐补充完整，有问题的地方请大佬多多指教）