> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_53377952/article/details/136014591)

#### 1 前言

.  
【文章首发于：https://mp.weixin.qq.com/s/bAYphUPNG1gwsOugReFSbg】

今天在 [bugku](https://so.csdn.net/so/search?q=bugku&spm=1001.2101.3001.7020) 上做了几道 Misc 题，都是和流量相关的，做题顺序也恰好是从简单到难，也是学到不少东西

分别是：USB 流量截取 -->> 花点流量听听歌

![](https://img-blog.csdnimg.cn/img_convert/e18b2c8451a647d255aa779af644d335.png)

![](https://img-blog.csdnimg.cn/img_convert/c7fc33aa43a81c57ddd9e95af318a7fc.png)

遇到一些问题，这里做个记录，也因为第一次正式接触流量题，比较深奥的东西暂写不出，同时很多东西不太熟悉、懵懂懵懂，如笔记有错误还请不吝指正。

[本意不是写 wp，所以就不具体写解题]

#### 1.1 前置知识

​ USB 流量指的是 USB 设备接口的流量，攻击者能够通过监听 [usb 接口](https://so.csdn.net/so/search?q=usb%E6%8E%A5%E5%8F%A3&spm=1001.2101.3001.7020)流量获取键盘敲击键、鼠标移动与点击、存储设备的铭文传输通信、USB 无线网卡网络传输内容等等。在 CTF 中，USB 流量分析主要以键盘和鼠标流量为主。

###### ①键盘流量：

​ USB 协议键盘数据部分，会出现在 `Leftover Capture Data` 或者 `HID Data` 中（这就是踩的坑），数据长度为 8 个字节，我们关注第 3 个字节 `15`，这里是 16 进制的`15`. 每次键盘操作后都会产生一个数据包。

![](https://img-blog.csdnimg.cn/img_convert/8e148b932473efaf284ab17834ceb1d6.png)

​ 以及 `HID Data` 中的数据：

![](https://img-blog.csdnimg.cn/img_convert/2703ed927bb8c26aba71a5dae8d91ba6.png)

###### ②鼠标流量：

​ USB 协议鼠标数据部分，也会出现在 `Leftover Capture Data` 或者 `HID Data` 中，此时数据长度有所变化，为 4 个字节。（写此文的时候还未做过此类题型，直接拿的参考文章里面的图）

![](https://img-blog.csdnimg.cn/img_convert/5ea7fb333ac5e5f89dae6e930203c871.png)

参考：https://blog.csdn.net/ON_Zero/article/details/130528679

#### 2 正文

###### 题 1、USB 流量截图：

第一题很友好，入门 usb 流量分析的经典题。直接 wireshark 打开 usb 流量包，不需要做任何过滤器筛选。

数据包里面全有`Leftover Capture Data`，我们做相关提取就好

**注意下图共 66 条数据包**

![](https://img-blog.csdnimg.cn/img_convert/7a2d48372152f9806c9b9da6c3dc6ba1.png)

跟着 https://blog.csdn.net/ON_Zero/article/details/130528679 手敲了一遍代码，成功把`Leftover Capture Data`流量提取出来了

这里我们看上图，发现所有数据包都是 `Leftover Capture Data` ，于是可以跟着参考文章里面的 tshark 的这个命令提取数据：

```
.\tshark.exe -r D:\Downloads\usb.pcap -T fields -e usb.capdata >D:\Downloads\usbdata.txt
```

![](https://img-blog.csdnimg.cn/img_convert/5b3f95122c2612cd9535fb61e7e24f00.png)

提取出来的 usbdata.txt ，也有 66 条数据，同 pacp 里面数据条数一致。

![](https://img-blog.csdnimg.cn/img_convert/a8fffe2830af6f9e55b0c87421cf8417.png)

这里介绍一下我们做题时 **tshark** 的常用参数：【官方文档：https://www.wireshark.org/docs/man-pages/tshark.html】

> -r：要分析的输入文件，-r usb.pcag
> 
> -T：输出的格式，可以选择 **ek|fields|json|jsonraw|pdml|ps|psml|tabs|text** ，注意 fields 要配合 -e 参数使用。
> 
> -e：导出的协议字段，可以同时使用多个 -e，例如 **-e frame.number -e ip.addr**
> 
> -Y：显示过滤器，可以理解为 wireshark 的过滤器

`-r`：显示所有报文

![](https://img-blog.csdnimg.cn/img_convert/7fd301fa5956884069b5561616dd889e.png)

`-T fields`：不加 - e 无法使用

![](https://img-blog.csdnimg.cn/img_convert/8de3a6fb8f6a8c9daf088ff6f9ebfd40.png)

`-T fields -e frame.number -e usb.src`：只导出 f.num 和 usbsrc 这两个字段

![](https://img-blog.csdnimg.cn/img_convert/3e6dc7585d691294afbd1dc99e0800c6.png)

`-Y 'usb.data_len == 8 && usb.src=="2.5.1"'`：过滤条件为 data_len==8 且 src 地址为 "2.5.1"

（-Y 后面尽量外面用单引号）

![](https://img-blog.csdnimg.cn/img_convert/b2beac03ef2938aadeaa265c7ef499c8.png)

最后附此题脚本 1：

```
#脚本1
#python3
def setp1():
    f=open("usbdata.txt",'r')
    fi=open("usbdata_out.txt","w")
    while 1:
        a=f.readline().strip()
        # print(a)
        if a:
            if len(a)==16:
                out=''
                for i in range(0,len(a),2):
                    if i+2 != len(a):
                        out+=a[i]+a[i+1]+":"
                    else:
                        out+=a[i]+a[i+1]
                fi.write(out+"\n")
        else:
            break
    fi.close()

def step2():
    mappings = { 0x04:"A",  0x05:"B",  0x06:"C", 0x07:"D", 0x08:"E", 0x09:"F", 0x0A:"G",  0x0B:"H", 0x0C:"I",  0x0D:"J", 0x0E:"K", 0x0F:"L", 0x10:"M", 0x11:"N",0x12:"O",  0x13:"P", 0x14:"Q", 0x15:"R", 0x16:"S", 0x17:"T", 0x18:"U",0x19:"V", 0x1A:"W", 0x1B:"X", 0x1C:"Y", 0x1D:"Z", 0x1E:"1", 0x1F:"2", 0x20:"3", 0x21:"4", 0x22:"5",  0x23:"6", 0x24:"7", 0x25:"8", 0x26:"9", 0x27:"0", 0x28:"\n", 0x2a:"[DEL]",  0X2B:"    ", 0x2C:" ",  0x2D:"-", 0x2E:"=", 0x2F:"[",  0x30:"]",  0x31:"\\", 0x32:"~", 0x33:";",  0x34:"'", 0x36:",",  0x37:"." }
    keys = open("usbdata_out.txt",'r')
    nums=[]
    flag=''
    for key in keys:
        # print(key.strip())
        key=key.strip()
        tmp=key.split(":")
        # print(tmp)
        flags=True #标识符，识别其他字符是否全为 00
        for i in range(len(tmp)):
            if i!=2:
                if tmp[i]=="00":
                    continue
                else:
                    flags=False
                    break
        if flags:
            nums.append(int(key[6:8],16))
        pass

    for num in nums:
        # print(num)
        if num in mappings:
            # print(mappings[num])
            flag+=mappings[num]
    print(flag)
    # FLAGPR3550NWARDSA2FEE6E0
```

到这里一切正常，学着参考资料做出了这题，但下一题就发现了问题

###### 题 2：花点流量听听歌

拿到题目是一个压缩包，压缩包里面有一个 mp3 文件（mp3 可以正常播放），通过 binwalk、strings 等分析发现 mp3 文件里面还有隐写了其他压缩包

进一步提取得到 1、加密的 flag.rar 2、描述文件. txt 3、whereiskey.pcapng 流量包

![](https://img-blog.csdnimg.cn/img_convert/755b25844b2537a9d8aab2c1f3f8447f.png)

###### 分析流量包：

我们重点关注的是 3 流量包，打开分析：

![](https://img-blog.csdnimg.cn/img_convert/34241c2daef19f145b2e9ecbfb3dced5.png)

这次更复杂点，数据量更大，而且仔细一看能发现比之前的 Usb 的流量包多了不同的数据，

其中前 30 条报文应该是 usb 通信的准备配置数据包，往后应该就是鼠标 / 键盘通信的数据包了，仔细观察发现是 8 个字节，应该是键盘流量

![](https://img-blog.csdnimg.cn/img_convert/c31682e2d358cae9e572048fccab1a8e.png)

这时准备用之前的 tshark 语句继续导出：（兴高采烈）

```
tshark.exe -r whereiskey.pcap -T fields -e usb.capdata >D:\Downloads\usbdata.txt
```

奇怪的事情发生了，发现导出的 usbdata.txt 内容为空？后续换其他电脑其他版本的 wireshark，以及 kali 中的 wireshark 都无法提取

第一反应想到是否是因为 whereiskey.pcagpng 和 usb.pcag 内容有差异导致的，于是仔细对比两个数据包的差异：

> 相同处：
> 
> ​ 1、usb.pcag 和 whereiskey.pcag 都有从 ‘Source==x.x.x -->> Des == host’ 的数据包，其 info 信息都为 ‘URB_INTERRUPT IN’。同时 usb.pcag 的数据包全是这种流量。
> 
> 不同处：1、whereiskey.pcag 比 usb.pcag 多了配对的流量 (非重点)
> 
> ​ 2、whereiskey 包还有 从 ‘Source==host -->> Des == x.x.x’ 的数据包（会是这两个的原因吗）
> 
> ​ 3、whereiskey 包的 ‘Source==x.x.x -->> Des == host’ 的框架不再是之前的 `Leftover Capture Data` 而是 `HID Data` （这也是后来才发现的，最初没注意到）

###### 提取生成 new.pcapng：

于是在我最初的想法中，1、2 是原因，所以这里我用了一个笨方法，手动从 wireshark 中提取之前那种数据包。好巧不巧算是瞎猫碰上死耗子，操作如下：

① 过滤器填写：usb.src==“2.5.1”，

② 导出筛选的 76 条数据包，保存为 new.pcapng

![](https://img-blog.csdnimg.cn/img_convert/00c55351f3f543c10291210b0b540862.png)

new.pcapng：（不仔细看没发现新生成的 new.pcapng 包里面已经从 之前的 HID Data 变 ---->>> 成 Left xxx 了）

![](https://img-blog.csdnimg.cn/img_convert/171ca6547c67960351b892c2d35c6270.png)

③ 使用命令：

```
tshark.exe -r new.pcap -T fields -e usb.capdata >D:\Downloads\usbdata.txt
```

咦，又可以了？

![](https://img-blog.csdnimg.cn/img_convert/48b86c55a4f356d1fb2775076695be7c.png)

当时隐约感觉有点问题，但是又没细想，继续写题了。回过头来越想越不对劲，于是先仔细看了看别人的 wp，附两篇 wp：

1、https://demonstaralgol.github.io/Misc/Bugku/Bugku%20Misc%2012%EF%BC%8869-72%EF%BC%89.html

2、https://www.cnblogs.com/Moomin/p/15085041.html

发现别人都是用 工具 [UsbKeyboardDataHacker.py](https://github.com/WangYihang/UsbKeyboardDataHacker/blob/master/UsbKeyboardDataHacker.py) 直接一步到位的，于是 github 查看 py 源码，发现里面大致和之前写的代码逻辑一样，最不一样之处就在于工具将 tshark 命令写在了 python 脚本里面执行，

![](https://img-blog.csdnimg.cn/img_convert/b8679023caf1fbc3288a164e1e406bd4.png)

又在 kali 上执行这个脚本发现还是无法提取 whereiskey.pcap 的流量，于是将里面 `tshark` 的语句提取出来，直接拿到命令行使用，发现仍有问题，经过不断控制变量测试，最终发现是 -e usb.capdata 搞的鬼。

（ps 拓展一下：[【已解决】Python 的坑：os.system() 运行带有空格的长路径和双引号参数有 bug_os.system 怎么调试 - CSDN 博客](https://blog.csdn.net/Scott0902/article/details/129446831) ）

测试过程也遇到了’脚本 / issues/7’中的情况，这个 - Y 参数值得注意，也更加肯定了我要重新读 tshark 的帮助文档，要搞清楚为什么语法正确但提取失败。

![](https://img-blog.csdnimg.cn/img_convert/b417721562a37cc26b506ba1fcddc632.png)

再去查阅 tshark --help 、官方文档、以及其他参考资料：

1、[一文读懂网络报文分析神器 Tshark](https://cloud.tencent.com/developer/article/2312883)：https://cloud.tencent.com/developer/article/2312883

2、[USB HID 流量分析 - LinuxStory](https://linuxstory.org/usb-hid-traffic-analysis/)：https://linuxstory.org/usb-hid-traffic-analysis/

3、[CTF 中我的 USB 键盘鼠标流量解密指南和脚本 - FreeBuf](https://www.freebuf.com/sectool/347971.html)：https://www.freebuf.com/sectool/347971.html

4、https://www.wireshark.org/docs/man-pages/tshark.html

5、https://wiki.wireshark.org/CaptureSetup/USB#a-word-of-warning-about-usbpcap

经过一系列的测试，查阅、最终发现 tshark 的 -e 参数是显示字段的功能，-Y 起过滤条件的作用。

这里对 whereiskey.pcapng 包使用 -e usb.capdata 参数的话，导出为空并不是语法错了，其语法是正确的（tip：理由是在 wireshark 的过滤条件里面写 "usb.capdata" 并没有爆红色，而是绿色），而是因为 whereiskey.pcapng 本身就没有 usb.capdata 字段。于是写了几个语法来佐证我的想法：

```
1、输入：tshark -r whereiskey.pcapng -T fields -e frame.number
# 因为frame.number是肯定存在的，就用frame.number来测试
# 正常返回所有序列号，1-182，对应数据包条数
```

![](https://img-blog.csdnimg.cn/img_convert/15c6ee85579c736aa9ffe0381b8eddb4.png)

```
2、输入：tshark -r whereiskey.pcapng -T fields -e frame.number -Y "usb.capdata && usb.data_len == 8"
# 这里参考的是 UsbKeyboardDataHacker.py 的issues
# 发现查询为空，于是改
3、输入：tshark -r whereiskey.pcapng -T fields -e frame.number -Y "usb.data_len == 8"
# 有数据了，于是这才发现是usb.capdata根本不存在
```

![](https://img-blog.csdnimg.cn/img_convert/098f76a0a53e734c4e29aba6006419fe.png)

那么这个 usb.capdata 究竟是个啥呢？先回到两个 wireshark 数据包里面看看：

![](https://img-blog.csdnimg.cn/img_convert/ae055d03109852439fe87133adff60cc.png)

![](https://img-blog.csdnimg.cn/img_convert/9933539e04f418fd16a5da358fb4d763.png)

发现了啥？whereiskey 包在 “usb.capdata” 这一过滤条件下显示 0/182，但 new.pcapng 包，是 76/76，全部显示的？

new.pcag 是从 whereiskey.pcag 提取出来的啊，既然我都能在 new.pcag 里面用 usb.capdata 成功筛选出数据，为啥不能在 whereiskey.pcag 里面筛选呢，带着这个疑问仔细比对两个数据包：

```
根据红框可以确定new-76和whereiskey-181是同一个数据包，也即76从181提取过来的

那为什么无法提取whereiskey-181呢？

于是发现了绿框中的内容：一个是 HID Data / 另一个是Leftover Capture Data
```

![](https://img-blog.csdnimg.cn/img_convert/68c5929f72fc796971e8c73dc45bab73.png)

难怪。。这下就明确了，再次 wireshark 查看：

new.pcapng：由这里得出提取 `Leftover Capture Data` 的语句应为：`usb.capdata`。

【这里就对应上之前说的 “瞎猫碰上死耗子”，即对 new.pcapng 使用同样的语句就能提取成功，反观 whereiskey.pcag 不行】

![](https://img-blog.csdnimg.cn/img_convert/9a778074bf4d160cb915f769fdd40b79.png)

whereiskey.pcag：这里得提取 `HID Data` 的语句应为：`usbhid.data`

![](https://img-blog.csdnimg.cn/img_convert/98d9a873891cc5ee3ccbcb28b8fccd85.png)

至此，问题得到解决：# 重新更新语句：

```
tshark -r whereiskey.pcapng -T fields -e usbhid.data -Y 'usb.data_len == 8 && usb.src=="2.5.1"'
```

![](https://img-blog.csdnimg.cn/img_convert/0c6f0211e753a1056a0606d7780a06bd.png)

再次运行脚本 1，得到：

THK[DEL]EPE[DEL]ASY[DEL]SWOI[DEL]RDS[DEL]NOTU[DEL]HES[DEL]REB[DEL]6[DEL]

ok 完美解决。

最后，建议自己重新更新一下脚本，多写几个判断情况，下次遇到这种题，你也能秒出 flag，虽然也比手动提取快不了几十秒哈哈哈

#### 3 参考资料：

1.[USB HID 流量分析]，https://www.freebuf.com/sectool/347971.html

2.[CTF 中我的 USB 键盘鼠标流量解密指南和脚本]，https://www.freebuf.com/sectool/347971.html

3.[【流量分析】USB 键盘与鼠标流量分析]，https://blog.csdn.net/ON_Zero/article/details/130528679

4.[tshark 官方帮助文档]，https://www.wireshark.org/docs/man-pages/tshark.html

5.[一文读懂网络报文分析神器 Tshark： 100 + 张图、100 + 个示例轻松掌握]，https://cloud.tencent.com/developer/article/2312883

6.[Bugku-Misc - 花点流量听听歌]，https://www.cnblogs.com/Moomin/p/15085041.html

7.[Bugku Misc 12（69-72）]，https://demonstaralgol.github.io/Misc/Bugku/Bugku%20Misc%2012%EF%BC%8869-72%EF%BC%89.html

8.[深入理解 USB 流量数据包的抓取与分析]，https://www.cnblogs.com/ECJTUACM-873284962/p/9473808.html

9.UsbKeyboardDataHacker.py，https://github.com/WangYihang/UsbKeyboardDataHacker/blob/master/UsbKeyboardDataHacker.py

10.[Python 的坑: os.system() 运行带有空格的长路径和双引号参数有 bug]，https://blog.csdn.net/Scott0902/article/details/129446831

11.[Wireshark 全版本下载地址](https://blog.csdn.net/qq_40233736/article/details/103928295)：https://1.as.dl.wireshark.org/win64/all-versions/

#### 后记：

因为在找寻问题的过程中我也不可能就能肯定可以解决它，复现这一发现过程是凭记忆，其中难免会因为记忆混乱，有一些重复或者顺序上的错误，请体谅。最主要是将这一发现过程分享出来，供大家参考，加深印象。

对于比较熟悉这一题型的大佬应该是不会看到这篇文章，不过对于我们菜鸟来说，我们做题目的时候遇到无法直接复现别人的 wp 中部分步骤时，可以参考我这种思路来找、解决问题。为啥别人可以直接脚本一步出来结果，一种是 usb 协议改变了，或者本机 wireshark 版本、环境有问题。二就是大概率其中的一些坑他人也踩过、于是后来已经修改过代码，再次遇到这种题就直接脚本一键出来结果了。所以我们复现遇到问题，也不妨多看看多想想，总能找到解决方法的（）

两个小问题：

①为啥 whereiskey.pcapng 经过筛选重新导出了 new.pcapng 后，new.pcapng 里面从一开始的 `Leftover Capture Data` 变成`HID Data` 了呢？我的猜测是 usb 协议更新了，后续支持经过后者传输数据。猜测理由之一是数据包时间显示是 2021 年的，但现在已经 2024 年了，很多东西已经更新了，经过我在 wireshark 里面重新导出后，wireshark 已经自动把 `Leftover Capture Data` 变成 `HID Data` 了。

②另一个有趣的事，我把 whereiskey.pcapng（或者 old.pcapng） 和 new.pcapng 一齐发给学姐看了，在她电脑上居然可以直接 usb.capdata 搜到【汗颜】，她的 wireshark 版本是：3.6.7.0

![](https://img-blog.csdnimg.cn/img_convert/39352219286d583e95d9a4d5e9b9ea59.png)

![](https://img-blog.csdnimg.cn/img_convert/773ff777eea3b17c57087e6595eab7a4.png)

为此，我还特意重新用 kali/win10，并且试过了 3.6.7.0，3.6.8.0，以及截至到 2024-2-2 最新版本的 wireshark，都不能直接通过这个过滤。

细心点可以发现她给的 old2.pcapng 应该不是我发的原文件，因为原来是 182 条，但不知为何总条数只有 91 条了。但为啥又凑巧是一半呢… 我也不懂，最后我的解释是：

![](https://img-blog.csdnimg.cn/img_convert/e403ac725b2b3e5fe4dc39be43d96b3a.png)