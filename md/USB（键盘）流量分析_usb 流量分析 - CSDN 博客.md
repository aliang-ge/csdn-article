> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/m0_64702918/article/details/131759542)

#### 标题

*   [找到有用的 usb 流量](#usb_2)
*   *   [鼠标流量](#_4)
    *   [键盘流量（自己做题收获，可能仅适用本题）](#_12)
    *   *   [脚本或者自己对照](#_19)
*   [USB（键盘）流量分析具体流程](#USB_34)

hws2023 的一道 misc, 没接触过，写一份博客，稍微带一点鼠标流量，自己做题收获，可能在某些情况仅适用本题

找到有用的 usb 流量
------------

网上大部分讲 [USB 协议](https://so.csdn.net/so/search?q=USB%E5%8D%8F%E8%AE%AE&spm=1001.2101.3001.7020)数据在 **Leftover Capture Data 域**中，就这道题，以及目前已知少量博客显示，**HID Data** 域中也具有价值（USB URB 里找有没有数据就行）。鼠标流量数据长度为四个字节，键盘流量数据长度为八个字节，就这个题而言，若鼠标流量数据长度不满足，格式也不太对

### 鼠标流量

> 第一个字节：代表按键（00 时, 代表没有按键；01 时, 代表按左键；02 时, 代表当前按键为右键）  
> 第二个字节：值为正时，代表鼠标右移像素位；  
> 值为负时，代表鼠标左移像素位  
> 第三个字节：代表垂直上下移动的偏移（当值为正时，代表鼠标上移像素位；值为负时，代表鼠标下移像素位）  
> 上述引用和进一步了解后续鼠标流量具体流程可参考这篇文章：https://blog.csdn.net/qq_46150940/article/details/115431953

![](https://img-blog.csdnimg.cn/46cbbfce1c38400c8732a3a451ce1f56.png)

### 键盘流量（自己做题收获，可能仅适用本题）

本题就在 [HID](https://so.csdn.net/so/search?q=HID&spm=1001.2101.3001.7020) Data 域

> 第一个字节：代表按键（00 时, 代表没有按键；不论 02 或者 20 做题时统一当 shift 键）  
> 第三个字节：代表键盘敲击时具体字母（[hid 键盘报告格式](https://blog.csdn.net/a65135793/article/details/80287250?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-80287250-blog-109996122.235%5Ev38%5Epc_relevant_sort&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-80287250-blog-109996122.235%5Ev38%5Epc_relevant_sort&utm_relevant_index=2)）

![](https://img-blog.csdnimg.cn/c4407b18c0a143d5a1ca05603037c5dc.png)

#### 脚本或者自己对照

找出相应信息后，脚本直接还原信息比较快速方便，但是如果刚刚接触，一下子拿一个现成的脚本去改，不理解，修改难度比较大，就可以先找一个差不多脚本运行一下（得出结果不全，但是可以给手动对照一点参考，自己理解对照的对不对），然后自己手动对照 hid 键盘报告格式，了解原理，改着也方便  
取一小段：

```
02:00:00:00:00:00:00:00   // 第三个字节没有数据不看
02:00:04:00:00:00:00:00   // 第一个字节为02，键盘操作shift键和第三个字节代表的键
//由上边可得第三个字节没有数据的可以直接忽略，即第一个字节为02或者20，shift键只作用于本行
20:00:26:00:00:00:00:00   // 第一个字节为20，shift键和第三个字节代表的键

20:00:00:00:00:00:00:00
00:00:00:00:00:00:00:00
00:00:10:00:00:00:00:00   // 直接第三个字节代表的键
00:00:00:00:00:00:00:00
```

USB（键盘）流量分析具体流程
---------------

流量包直接导入即可，寻找有用信息的 IP，过滤导出

```
usb.src =="2.3.1"
```

![](https://img-blog.csdnimg.cn/dd3bb4831fc04d959ae7d1cffe452f7b.png)  
拖 kali 里面 (python 运行方便，也不咋出错)，拖不进去的配一下 tools。**python 脚本在那块，就在哪块，右键打开终端！！！不容易出错**  
![](https://img-blog.csdnimg.cn/c702c9a97bce4ecabf0fd90ea7c4566a.png)  
使用 tshark 命令把 pcap 的数据提取并去除空行到 usbdata.txt

```
# sed '/^\s*$/d' 剔除空行,减少不必要麻烦特别鼠标流量
tshark -r usb.pcapng -T fields -e usb.capdata | sed '/^\s*$/d' > usbdata.txt
```

对！没错，我改包名了，不太会 tshark 命令，就和找到教程里面的包名保持一致，还是要了解一下的。[可以看这个！](https://www.cnblogs.com/liun1994/p/6142505.html)

> **注意大小写**  
> -r: 读取本地文件，可以先抓包存下来之后再进行分析;  
> -T 设置解码结果输出的格式，包括 text,ps,psml 和 pdml，默认为 text。  
> -t 设置解码结果的时间格式。“ad” 表示带日期的绝对时间，“a” 表示不带日期的绝对时间，“r” 表示从第一个包到现在的相对时间，“d” 表示两个相邻包之间的增量时间（delta）。  
> -E: 当 - T 字段指定时，设置输出选项，header=y 意思是头部要打印;  
> -e: 当 - T 字段指定时，设置输出哪些字段

```
# 字节和字节间加冒号的脚本
f=open('usbdata.txt','r')
fi=open('out.txt','w')
while 1:
    a=f.readline().strip()
    if a:
        if len(a)==16: # 鼠标流量的话len改为8
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

还原信息（有冒号时提取数据的 [6:8]，无冒号时数据在 [4:6]）

```
# x
normalKeys = {
    "04":"a", "05":"b", "06":"c", "07":"d", "08":"e",
    "09":"f", "0a":"g", "0b":"h", "0c":"i", "0d":"j",
     "0e":"k", "0f":"l", "10":"m", "11":"n", "12":"o",
      "13":"p", "14":"q", "15":"r", "16":"s", "17":"t",
       "18":"u", "19":"v", "1a":"w", "1b":"x", "1c":"y",
        "1d":"z","1e":"1", "1f":"2", "20":"3", "21":"4",
         "22":"5", "23":"6","24":"7","25":"8","26":"9",
         "27":"0","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t",
         "2c":"<SPACE>","2d":"-","2e":"=","2f":"[","30":"]","31":"\\",
         "32":"<NON>","33":";","34":"'","35":"<GA>","36":",","37":".",
         "38":"/","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>",
         "3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>",
         "44":"<F11>","45":"<F12>"}
# shift+x
shiftKeys = {
    "04":"A", "05":"B", "06":"C", "07":"D", "08":"E",
     "09":"F", "0a":"G", "0b":"H", "0c":"I", "0d":"J",
      "0e":"K", "0f":"L", "10":"M", "11":"N", "12":"O",
       "13":"P", "14":"Q", "15":"R", "16":"S", "17":"T",
        "18":"U", "19":"V", "1a":"W", "1b":"X", "1c":"Y",
         "1d":"Z","1e":"!", "1f":"@", "20":"#", "21":"$",
          "22":"%", "23":"^","24":"&","25":"*","26":"(","27":")",
          "28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":"<SPACE>",
          "2d":"_","2e":"+","2f":"{","30":"}","31":"|","32":"<NON>","33":"\"",
          "34":":","35":"<GA>","36":"<","37":">","38":"?","39":"<CAP>","3a":"<F1>",
          "3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>",
          "41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}
output = []
keys = open('out.txt')
for line in keys:
    try:
#通过line[]取值过滤无用信息，注意(line[0]!='0' and line[0]!='2') or (line[1]!='0' and line[1]!='2') ，一般情况x不等于信息字符位其余的位line[x]!='0'即可
        if (line[0]!='0' and line[0]!='2') or (line[1]!='0' and line[1]!='2') or line[3]!='0' or line[4]!='0' or line[9]!='0' or line[10]!='0' or line[12]!='0' or line[13]!='0' or line[15]!='0' or line[16]!='0' or line[18]!='0' or line[19]!='0' or line[21]!='0' or line[22]!='0' or line[6:8]=="00":
             continue
        if line[6:8] in normalKeys.keys():
#由于02，20两种情况，output += 不能写一行，加一个if语句
            if line[0]=='2':
            		output += [[normalKeys[line[6:8]]],[shiftKeys[line[6:8]]]][line[0]=='2']
            else:
            		output += [[normalKeys[line[6:8]]],[shiftKeys[line[6:8]]]][line[1]=='2']
        else:
#对照的很全了，基本可以忽略[unknown]
            output += ['[unknown]']
    except:
        pass

keys.close()

flag=0
print("".join(output))
for i in range(len(output)):
    try:
        a=output.index('<DEL>')
        del output[a]
        del output[a-1]
    except:
        pass

for i in range(len(output)):
    try:
        if output[i]=="<CAP>":
            flag+=1
            output.pop(i)
            if flag==2:
                flag=0
        if flag!=0:
            output[i]=output[i].upper()
    except:
        pass
# 注意版本，有的为：print 'output :' + "".join(output)
print ('output :' + "".join(output))
```

运行可得（运行时由于 tab 键和空格混用，可能会报错）  
![](https://img-blog.csdnimg.cn/2b5c493e48fd43fea583bcfcb5acdc02.png)  
放大一点  
![](https://img-blog.csdnimg.cn/8a1f129bcc1b445c8317341328dc3d61.png)  
解释一下，运行完的第一行，每一行还原后的结果，看不懂 output：这一行怎么来的，自己键盘敲一遍就行（`<DEL>删除前一个，<CAP> <CAP>之间的大写`）[unknown] 直接删就行，前提需要确保脚本的正确，以及对照数据的完整性（例如：例子脚本中的 normalKeys 和 shiftKeys 数据）

流程学习于：https://blog.csdn.net/qq_43625917/article/details/107723635