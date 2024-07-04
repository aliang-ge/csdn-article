> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.51cto.com](https://www.51cto.com/article/526928.html)

> USB 接口是目前最为通用的外设接口之一，通过监听该接口的流量，可以得到很多有意思的东西，例如键盘击键，鼠标移动与点击，存储设备的明文传输通信、USB 无线网卡网络传输内容等。

USB 接口是目前最为通用的外设接口之一，通过监听该接口的流量，可以得到很多有意思的东西，例如键盘击键，鼠标移动与点击，存储设备的明文传输通信、USB 无线网卡网络传输内容等。本文将通过两个 CTF 题，讲解如何捕获 USB 接口的数据，以及键盘鼠标 USB 协议的具体解析方式。

一、简介

USB 接口是目前最为通用的外设接口之一，通过监听该接口的流量，可以得到很多有意思的东西，例如键盘击键，鼠标移动与点击，存储设备的明文传输通信、USB 无线网卡网络传输内容等。本文将通过两个 CTF 题，讲解如何捕获 USB 接口的数据，以及键盘鼠标 USB 协议的具体解析方式。相关下载链接：[http://pan.baidu.com/s/1i57b33B](http://pan.baidu.com/s/1i57b33B)

**二、 USB capture**

USB 流量的捕获可以使用 wireshark 或者 usbpcap 来进行，最新版本的 wireshark 已经支持 USB 接口的捕获，且在安装时会提示 usbpcap 的安装，当前网上已有相关中文资料对 wireshark 抓取 usb 数据包的方法进行讲解，感兴趣的读者可阅读参考链接，在此我们使用一种相对简单的方式，即直接采用 usbpcap 捕获 USB 流量。该软件的下载地址为 [http://desowin.org/usbpcap/](http://desowin.org/usbpcap/)，可支持 32 位以及 64 位的 winxp 至 win10 操作系统，安装完成后须重启机器或者按照提示选择重启所有 USB 设备。

安装完成后，直接双击 USBPcapCMD.exe，按照提示信息选择 filter，输入文件名，便可愉快地等产生的信息被捕获了，程序运行界面如下图：

![](http://s3.51cto.com/wyfs02/M02/8C/68/wKiom1hrVeWAgDl_AACaxZyYpkU495.jpg)

**三、键盘流量解析**

以今年 xnuca misc 专场的 old years 题为例，该题目给出一个 pcap 包，使用 wireshark 打开，看到 Protocol 为 USB 协议。

USB 协议的数据部分在 Leftover Capture Data 域之中，右键 leftover capture data –> 应用为列，可以将该域的值在主面板上显示，键盘数据包的数据长度为 8 个字节，击键信息集中在第 3 个字节，每次 key stroke 都会产生一个 keyboard event usb packet ，如下图：

![](http://s3.51cto.com/wyfs02/M00/8C/68/wKiom1hrViLjRaVtAADM-kqZr18082.jpg)

网上查找 USB 协议的文档，可以找到这个值与具体键位的对应关系，http://www.usb.org/developers/hidpage/Hut1_12v2.pdf

第 53 页有一个 usb keyboard/keypad 映射表：

![](http://s4.51cto.com/wyfs02/M00/8C/64/wKioL1hrVjriU5OwAACJjaJSQSg944.jpg)

使用 wireshark 自带的 tshark 命令行工具，可以将 leftover capture data 单独提取出来，具体命令为：

```
tshark.exe -r usb1.pcap -T fields -e usb.capdata > usbdata.txt
```

然后我们需要编写脚本从得出的 userdata.txt 文件中过滤出键盘击键相关的流量，并根据上述映射表，将键盘按键按照对应关系输出出来，这里附上简要的脚本：

```
mappings = { 0x04:"A",  0x05:"B",  0x06:"C", 0x07:"D", 0x08:"E", 0x09:"F", 0x0A:"G",  0x0B:"H", 0x0C:"I",  0x0D:"J", 0x0E:"K", 0x0F:"L", 0x10:"M", 0x11:"N",0x12:"O",  0x13:"P", 0x14:"Q", 0x15:"R", 0x16:"S", 0x17:"T", 0x18:"U",0x19:"V", 0x1A:"W", 0x1B:"X", 0x1C:"Y", 0x1D:"Z", 0x1E:"1", 0x1F:"2", 0x20:"3", 0x21:"4", 0x22:"5",  0x23:"6", 0x24:"7", 0x25:"8", 0x26:"9", 0x27:"0", 0x28:"\n", 0x2a:"[DEL]",  0X2B:"    ", 0x2C:" ",  0x2D:"-", 0x2E:"=", 0x2F:"[",  0x30:"]",  0x31:"\\", 0x32:"~", 0x33:";",  0x34:"'", 0x36:",",  0x37:"." } 
nums = [] 
keys = open('usbdata.txt') 
for line in keys: 
    if line[0]!='0' or line[1]!='0' or line[3]!='0' or line[4]!='0' or line[9]!='0' or line[10]!='0' or line[12]!='0' or line[13]!='0' or line[15]!='0' or line[16]!='0' or line[18]!='0' or line[19]!='0' or line[21]!='0' or line[22]!='0': 
         continue 
    nums.append(int(line[6:8],16)) 
keys.close() 
output = "" 
for n in nums: 
    if n == 0 : 
        continue 
    if n in mappings: 
        output += mappings[n] 
    else: 
        output += '[unknown]' 
print 'output :\n' + output
```

运行该脚本，便可得到输出结果如下：

![](http://s4.51cto.com/wyfs02/M00/8C/68/wKiom1hrV4fS0gpPAAAU3zURdjg222.png)

这里还有最后一个小弯，由提示中的 “不使用拼音” 等信息推测出上文的特殊编码可能是某种拼音之外的古老的输入法，例如五笔，毕竟这也是做 keylogger 的人需要考虑而且头疼的一个地方。尝试对照着输入，可以得出如下文字：

最后得出 flag . xnuca{wojiushifulagehaha}

**四、鼠标流量解析**

这是 xnuca 第二道 usb 流量的题，首先可以直接使用上述的解决方案试一下，得到这样一话：

解决本题需要把鼠标流量还原出来，然而鼠标与键盘不同，鼠标移动时表现为连续性，与键盘击键的离散性不一样，不过实际上鼠标动作所产生的数据包也是离散的，毕竟计算机表现的连续性信息都是由大量离散信息构成的。

首先同样使用 tshark 命令将 cap data 提取出来:

1tshark.exe -r usb2.pcap -T fields -e usb.capdata > usbdata.txt

![](http://s1.51cto.com/wyfs02/M02/8C/64/wKioL1hrV5Hzg6MXAAAMF0u4bIw360.png)

每一个数据包的数据区有四个字节，第一个字节代表按键，当取 0x00 时，代表没有按键、为 0x01 时，代表按左键，为 0x02 时，代表当前按键为右键。第二个字节可以看成是一个 signed byte 类型，其最高位为符号位，当这个值为正时，代表鼠标水平右移多少像素，为负时，代表水平左移多少像素。第三个字节与第二字节类似，代表垂直上下移动的偏移。

了解协议相关约定之后，可编写脚本将数据包的内容变成一系列点的集合，为了区分左右按键，可以特意对第一个字节的内容作一下判断。相关脚本如下：

```
nums = [] 
keys = open('data.txt','r') 
posx = 0 
posy = 0 
for line in keys: 
    if len(line) != 12 : 
         continue 
    x = int(line[3:5],16) 
    y = int(line[6:8],16) 
    if x > 127 : 
        x -= 256 
    if y > 127 : 
        y -= 256 
    posx += x 
    posy += y 
    btn_flag = int(line[0:2],16)  # 1 for left , 2 for right , 0 for nothing 
    if btn_flag == 1 : 
        print posx , posy 
keys.close()
```

本题的 flag 藏在右键信息中，当 btn_flag 取 2 时，运行脚本可以得到一系列坐标点：

![](http://s5.51cto.com/wyfs02/M01/8C/64/wKioL1hrV5Sxh4D5AAAnL6KAJ14579.png)

得到这些点之后，需要将他们画出来，因而需要辅以 gnuplot 或者其他的绘图工具，gnuplot 的命令为 "plot inputfile"，运行如下：

![](http://s4.51cto.com/wyfs02/M00/8C/68/wKiom1hrVwPw32sPAABNopf8YYQ533.jpg)

最后得到本题的 flag： XNUCA{USBPCAPGETEVERYTHING}

![](http://s5.51cto.com/wyfs02/M00/8C/64/wKioL1hrVxrSwLa5AAAi0HwB3HM965.jpg)