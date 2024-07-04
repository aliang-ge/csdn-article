> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_41225906/article/details/126425680)

#### 目录

*   [一、附加字符串](#_2)
*   [二、修改图片的宽高](#_7)
*   [三、jphide 图片隐写](#jphide_49)
*   [四、OutGuess 隐写](#OutGuess_68)
*   [五、异或隐写](#_82)
*   [六、盲水印](#_96)
*   [七、二维码画图](#_104)
*   [八、灰度图片 LSB 隐写](#LSB_149)

一、附加字符串
-------

flag 藏在图片中，使用 WinHex 软件或者记事本打开，搜索发现 flag。

![](https://img-blog.csdnimg.cn/7396784704054d9184124cf777d509ee.png)

二、修改图片的宽高
---------

![](https://img-blog.csdnimg.cn/4cb0ff5093614987bffd8de13d0f06dc.png)

像这种图片很明显是被修改过高度的，我们使用 python 脚本得出正常图片的宽高。

```
import zlib
import struct

filename = '1.png'
with open(filename, 'rb') as f:
    all_b = f.read()
    crc32key = int(all_b[29:33].hex(),16)
    data = bytearray(all_b[12:29])
    n = 4095  
    for w in range(n):          
        width = bytearray(struct.pack('>i', w))     
        for h in range(n):
            height = bytearray(struct.pack('>i', h))
            for x in range(4):
                data[x+4] = width[x]
                data[x+8] = height[x]
            crc32result = zlib.crc32(data)
            if crc32result == crc32key:
                print("宽为：",end="")
                print(width)
                print("高为：",end="")
                print(height)
                exit(0)
```

得出的结果是：

![](https://img-blog.csdnimg.cn/ee9a9ec573554156982e9fd289f178a0.png)

使用软件 WinHex 打开那张图片，就能看到当前的图片的高是不对的：

![](https://img-blog.csdnimg.cn/feb0d68efd68468a802813e6f425988e.png)

在 WinHex 里把高改成 01 df 就可以了，得到正常图片：

![](https://img-blog.csdnimg.cn/039477f388914162b9b2bce41241df50.png)

三、jphide [图片隐写](https://so.csdn.net/so/search?q=%E5%9B%BE%E7%89%87%E9%9A%90%E5%86%99&spm=1001.2101.3001.7020)
-------------------------------------------------------------------------------------------------------------

使用小程序 stegdetect 来检测图片是不是用了 jphide 隐写。  
在 cmd 中使用这个代码：`stegdetect.exe -tjopi -s 10.0 jphide.jpg`

![](https://img-blog.csdnimg.cn/22bb3d501af546acbc389186498f8fc7.png)

密码破解使用字典，代码为`stegbreak.exe -r rules.ini -f password.txt jphide.jpg`，得到密码为 power123

![](https://img-blog.csdnimg.cn/725ef52fde204f8bb747b43fa5c41b35.png)

再使用软件 Jphswin 来破解图片：

打开这个图片：  
![](https://img-blog.csdnimg.cn/a7482ca18c1447a394fb3f120855f92d.png)  
点击 Seek，输入破解到的密码，再另存为 txt 文件就能得到 flag。  
![](https://img-blog.csdnimg.cn/d4fdfea8f79448c2acba87f9f09a82c2.png)![](https://img-blog.csdnimg.cn/ca783ee71fcc47438e7c3e99242a8fc0.png)

四、OutGuess 隐写
-------------

需要在 kali 虚拟机里面使用 outguess 工具。

安装方法是：

```
git clone https://github.com/crorvick/outguess
# 进入outguess的目录
./configure && make && make install
```

一般图片属性 - 详细信息 - 备注里面会有图片的 Key  
![](https://img-blog.csdnimg.cn/68f587570db549478cea81013408a4f9.png)  
把图片文件复制到虚拟机里面，然后使用代码`outguess -k gUNrbbdR9XhRBDGpzz -r outguess.jpg -t 1.txt`能得到结果文件。

五、[异或](https://so.csdn.net/so/search?q=%E5%BC%82%E6%88%96&spm=1001.2101.3001.7020)隐写
------------------------------------------------------------------------------------

![](https://img-blog.csdnimg.cn/82daf0a08fa74d6fa7cb2d09900bdc9c.png)

这个图片需要用 Stegsolve 这个工具进行反色，点击上面`“>”`按钮得到：  
![](https://img-blog.csdnimg.cn/f032365a9e144214ba726adf96f1ed59.png)

再把下面的图片进行异或处理  
![](https://img-blog.csdnimg.cn/bfe57b1f69604e8ead909a54dac110ce.png)  
用这个 Image Combiner 功能  
![](https://img-blog.csdnimg.cn/a252dad97802479984c2fb68f3773b19.png)

最后用工具 QR Research 进行扫描，就可以得到结果  
![](https://img-blog.csdnimg.cn/eec696acb43540d595ec34687f8cc50b.png)

六、盲水印
-----

![](https://img-blog.csdnimg.cn/444822ef727b42b6a3105bcc3400357a.png)  
当一张图片的 WinHex 里面有这些东西，说明这个图片有盲水印，先在 kali 里面使用 binwalk，`binwalk -e 1.png --run-as=root`得到了很多文件。  
![](https://img-blog.csdnimg.cn/7acf42ca08d94bb1ba76f8583a02d642.png)  
把压缩文件复制到实体机，得到两张盲水印图片：  
![](https://img-blog.csdnimg.cn/1f8658bddc724192bccf2f91c1498503.png)  
使用`python bwmforpy3.py decode day1.png day2.png flag.png --oldseed`即可得到 flag 图片。

七、二维码画图
-------

![](https://img-blog.csdnimg.cn/ea57af3c0fa7429db331ec0affe6a356.png)  
有时候题目中会给这么一大串的 0 和 1 的组合，这时需要用到 python 脚本来进行画出二维码。

```
from PIL import Image
MAX = 60    #二维码长宽
pic = Image.new("RGB",(MAX, MAX))
str=" " #二进制数据
i=0
for y in range (0,MAX):
    for x in range (0,MAX):
        if(str[i] == '1'):
            pic.putpixel([x,y],(0, 0, 0))
        else:
            pic.putpixel([x,y],(255,255,255))
        i = i+1
pic.show()
#pic.save("flag.png")
```

*   如果电脑上没有安装 PIL 库，可以使用命令`pip install Pillow`来安装一下。  
    ![](https://img-blog.csdnimg.cn/94fd8dfc9c03417dacabee380e62bb52.png)  
    如果给的是这种坐标，也是需要用 python 脚本画图。

```
from PIL import Image, ImageDraw, ImageFont, ImageFilter
f1 = open(r'flag.txt','r')
width = 300
height = 300
image = Image.new('RGB', (width, height), (255, 255, 255))
draw = ImageDraw.Draw(image)
color = (0,0,0)
while 1:
    s = f1.readline()
    if not s:
        break
    s = s.strip('\n')
    s = s.lstrip('(')
    s = s.rstrip(')')
    a = int(s.split(',')[0],10)
    b = int(s.split(',')[1],10)
    draw.point((a, b), fill=color)
image.show()
```

八、灰度图片 LSB 隐写
-------------

如果一个题目给的是一张灰度图片，那么大概率是进行了 LSB 隐写，使用 python 脚本解码即可出现结果。

```
from PIL import Image
p = Image.open('1.png').convert('L')
a,b = p.size

flag = Image.new('L',(a,b),255)
for y in range(b):
    for x in range(a):
        if  p.getpixel((x,y))%2==0:
            flag.putpixel((x,y),255)
        else:
            flag.putpixel((x,y),0)
flag.show()
```