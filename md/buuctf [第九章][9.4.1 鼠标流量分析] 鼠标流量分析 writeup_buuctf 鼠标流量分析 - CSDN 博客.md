> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_49849300/article/details/132745735)

### 前置知识

参考 [USB 流量取证分析_usb 取证_lemonl1 的博客 - CSDN 博客](https://blog.csdn.net/qq_43431158/article/details/108717829 "USB流量取证分析_usb取证_lemonl1的博客-CSDN博客")

```
tshark -r usb.pcap -T fields -e usb.capdata > usbdata.txt
如果提取出来的数据有空行，可以将命令改为如下形式：
tshark -r usb2.pcap -T fields -e usb.capdata | sed '/^\s*$/d' > usbdata.txt
```

一般的鼠标流量都是四个字节

第一个字节代表按键：

当取 0x00 时，代表没有按键，当取 0x01 时，代表按左键，当取 0x02 时，代表当前按键为右键。

第二个字节可以看成是一个 signed [byte 类型](https://so.csdn.net/so/search?q=byte%E7%B1%BB%E5%9E%8B&spm=1001.2101.3001.7020)，其最高位为符号位：

当这个值为正时，代表鼠标水平右移多少像素 当这个值为负时，代表鼠标水平左移多少像素。

三个字节与第二字节类似，代表垂直上下移动的偏移。

![](https://img-blog.csdnimg.cn/4222a7ac652947198245a9ecdc5e7424.png)

### 解题

1. 得到 usbdata.txt

tshark -r atta3.pcapng -T [fields](https://so.csdn.net/so/search?q=fields&spm=1001.2101.3001.7020) -e usb.capdata | sed '/^\s*$/d' > usbdata3.txt

2. 得到 out.txt

```
f=open('usbdata.txt','r')
fi=open('out.txt','w')
while 1:
    a=f.readline().strip()
    if a:
        if len(a)==16: # 键盘流量len=16，鼠标流量len=8
            out=''
            for i in range(0,len(a),2):
                if i+2 != len(a):
                    out+=a[i]+a[i+1]+":"
                else:
                    out+=a[i]+a[i+1]
            fi.write(out)
            fi.write('\n')
    else:
        break
 
fi.close()
```

3. 转换为 xy 坐标

```
nums = []
keys = open('out.txt','r')
f = open('xy.txt','w')
posx = 0
posy = 0
for line in keys:
    if len(line) != 12 :
        continue
    x = int(line[3:5],16)
    y = int(line[6:8],16)
    if x > 127 :
        x -= 256
    if y > 127 :
        y -= 256
    posx += x
    posy += y
    btn_flag = int(line[0:2],16)  # 1 for left , 2 for right , 0 for nothing
    if btn_flag == 2 : # 1 代表左键
        f.write(str(posx))
        f.write(' ')
        f.write(str(posy))
        f.write('\n')
 
f.close()
```

4. 用 gnuplot 工具绘图

```
plot "xy.txt"
```

![](https://img-blog.csdnimg.cn/f1d9ef5142be451db89b82ea65bda13a.png)

```
flag{Hello}
```

![](https://img-blog.csdnimg.cn/9e8b037ad2f24736a226d78d8873e3fc.png)