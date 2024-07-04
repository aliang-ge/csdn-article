> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.51cto.com](https://www.51cto.com/article/614540.html)

> 通过对 USB 接口流量的监听，我们可以得到键盘的击键记录、鼠标的移动轨迹、磁盘的传输内容等一系列信息。

USB 是 UniversalSerial Bus(通用串行总线) 的缩写，是一个外部总线标准，用于规范电脑与外部设备的连接和通讯，例如键盘、鼠标、打印机、磁盘或网络适配器等等。通过对该接口流量的监听，我们可以得到键盘的击键记录、鼠标的移动轨迹、磁盘的传输内容等一系列信息。

在 Linux 中，可以使用 lsusb 命令，如图所示：

![](https://s1.51cto.com/oss/202004/16/ac9e9f024bd25575f59cbfb283f618fb.jpeg)

我们这里主要演示 USB 的鼠标流量和键盘流量。Linux 下的分析已经比较多了，下面的环境均在 Windows 下进行。

**一、鼠标流量**

**1.1 特点分析**

USB 鼠标流量的规则如下所示：

![](https://s1.51cto.com/oss/202004/16/530cf93e1f4c3b6fe6eb57522d77c936.jpeg)

1.2 使用 Wireshark 捕获和分析

要想使用 Wireshark 进行捕获，需要在安装时勾选上 usbpcap 工具选项，这样你的 Wireshark 中会有一个 usb 接口的选项，点击就可以进行抓包了。

![](https://s5.51cto.com/oss/202004/16/8add817f3fbbcd58ed759a3409a1e284.jpeg)

下图是我点击鼠标左键在屏幕上画圆圈的流量：

![](https://s2.51cto.com/oss/202004/16/5a8bf45f9e5d9a1c88fef4631263a899.jpeg)

有的鼠标可能协议不是很标准，会导致分析不了。

Wireshark 中捕获的 USB 流量集中在 Leftover Capture Data 模块，我们可以使用 tshark 工具来进行提取。在 Windows 中安装 tshark.exe 的目录中输入：

tshark.exe -r b.pcap -T fields -e usb.capdata >b.txt // 这里 b.pcap 是我抓捕的数据包名字，b.txt 是把提取的数据输入到 b.txt 中

![](https://s1.51cto.com/oss/202004/16/384b0031f66909c31199f2ff770491ea.jpeg)

查看 b.txt 的内容会发现，因为有的数据包无用所以出现了很多空行，在进行下一步之前，我们需要把空行去掉。

![](https://s2.51cto.com/oss/202004/16/f80642c9c96b35ff51700d0966669a42.jpeg)

把空行去掉之后，根据鼠标流量的规则绘制像素坐标，最后通过画图工具 (如 matlab 或者 python 的 matplotlib 进行绘制图像即可)。

了解原理之后，为了方便，可以直接使用王一航大佬的工具进行提取，输入：

```
python UsbMiceDataHacker.py b.pcap LEFT //其中b.pcap是我抓捕的数据包的名字
```

运行之后就可以看到画面：

![](https://s5.51cto.com/oss/202004/16/3c665920f4477335d4f0c1fa86d32e5d.jpeg)

需要注意的是这个工具必须在 python2 环境下，同时保证安装了 matplotlib 和 numpy。

**二、键盘流量**

**2.1 特点分析**

键盘数据包的数据长度为 8 个字节，击键信息集中在第 3 个字节，每次击键都会产生一个数据包。所以如果看到给出的数据包中的信息都是 8 个字节，并且只有第 3 个字节不为 0000，那么几乎可以肯定是一个键盘流量了。

在 USB 协议的 文档中搜索 keyboard。就可以找到击键信息和数据包中 16 进制数据的对照表：

![](https://s2.51cto.com/oss/202004/16/b6b85525ab89d6108a4a6042ac53e79a.png)

**2.2 使用 Wireshark 捕获和分析**

捕获的步骤与上面相似。下面以 XCTF 的高校战疫比赛中的一道例题 (ez_mem&usb) 来说明。

最后一步我们得到一个压缩包，通过密码进行解压后，得到一个键盘流量的文本文件：

![](https://s5.51cto.com/oss/202004/16/aafe69a257aab7876226d4c33f090f23.png)

根据键盘流量的特点，我们可以很容易判断出。

对照解码表使用代码进行提取即可，这里贴出代码：

```
# coding:utf-8 
import sys 
import os 
 
usb_codes = { 
    0x04: "aA", 0x05: "bB", 0x06: "cC", 0x07: "dD", 0x08: "eE", 0x09: "fF", 
    0x0A: "gG", 0x0B: "hH", 0x0C: "iI", 0x0D: "jJ", 0x0E: "kK", 0x0F: "lL", 
    0x10: "mM", 0x11: "nN", 0x12: "oO", 0x13: "pP", 0x14: "qQ", 0x15: "rR", 
    0x16: "sS", 0x17: "tT", 0x18: "uU", 0x19: "vV", 0x1A: "wW", 0x1B: "xX", 
    0x1C: "yY", 0x1D: "zZ", 0x1E: "1!", 0x1F: "2@", 0x20: "3#", 0x21: "4$", 
    0x22: "5%", 0x23: "6^", 0x24: "7&", 0x25: "8*", 0x26: "9(", 0x27: "0)", 
    0x2C: "  ", 0x2D: "-_", 0x2E: "=+", 0x2F: "[{", 0x30: "]}", 0x32: "#~", 
    0x33: ";:", 0x34: "'\"", 0x36: ",<", 0x37: ".>", 0x4f: ">", 0x50: "<" 
} 
 
 
def code2chr(filepath): 
    lines = [] 
    pos = 0 
    for x in open(filepath, "r").readlines(): 
        code = int(x[6:8], 16)  # 即第三个字节 
        if code == 0: 
            continue 
        # newline or down arrow - move down 
        if code == 0x51 or code == 0x28: 
            pos += 1 
            continue 
        # up arrow - move up 
        if code == 0x52: 
            pos -= 1 
            continue 
 
        # select the character based on the Shift key 
        while len(lines) <= pos: 
            lines.append("") 
        if code in range(4, 81): 
            if int(x[0:2], 16) == 2: 
                lines[pos] += usb_codes[code][1] 
            else: 
                lines[pos] += usb_codes[code][0] 
 
    for x in lines: 
        print(x) 
 
 
if __name__ == "__main__": 
    code2chr('E://CTF练习/杂项/18e4c103d4de4f07b33a42cb1f0eaa1d/00000122/usbdata.txt')
```

当然也可以直接使用王一航大佬的代码，直接从 pcap 包提取出文件，省去了中间很多步骤，代码也很通用。