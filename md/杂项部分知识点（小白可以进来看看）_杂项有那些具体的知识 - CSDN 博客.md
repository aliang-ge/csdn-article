> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/2201_75437983/article/details/135115203)

一、图片方面
------

### 图片隐藏

（1）如果遇到压缩包下载下来只有一张图片，我们第一反应就是先看它的属性，可能它的属性里面就藏着 flag，其次就是在[笔记本](https://so.csdn.net/so/search?q=%E7%AC%94%E8%AE%B0%E6%9C%AC&spm=1001.2101.3001.7020)里面打开它也有可能含有 flag，还有就是在 010editor 中查找 flag

![](https://img-blog.csdnimg.cn/direct/9b80fae0cd6141629d9f0fd1ea4c5b48.png)

![](https://img-blog.csdnimg.cn/direct/71cef36bc21e48e2b76a790c1b644f2a.png)

![](https://img-blog.csdnimg.cn/direct/f7c94a71e9ac4089b1c71024503c2bc0.png)

（2）如果前面三种情况都没有找到 flag 的话，我们首先要考虑的问题就是图片里面是否隐藏了压缩包，这个时候就要使用 [binwalk](https://so.csdn.net/so/search?q=binwalk&spm=1001.2101.3001.7020) 工具将压缩包分离出来。分离出来的压缩包内就会有 flag。

![](https://img-blog.csdnimg.cn/direct/9cbe9115a04f46fbaef79a2c18aef876.png)

![](https://img-blog.csdnimg.cn/direct/32dd14022d014f76bfc91757ef587afe.png)

![](https://img-blog.csdnimg.cn/direct/a37db016871c4afea8ce7d0fea607064.png)

（3）还有就是 outguess 隐藏，首先要根据不同的题先找到它的密码，其次再使用 outguess 来破解 flag

![](https://img-blog.csdnimg.cn/direct/4f0929d449094c36aaa7e500991fd6a7.png)

### 图片改变数据

 如果我们得到了一张图片，但是打不开，这个时候一般是它内部的数据出现了错误而导致打不开，我们可以使用 [010editor](https://so.csdn.net/so/search?q=010editor&spm=1001.2101.3001.7020) 打开图片，看其文件头，这个时候就发现它的文件头错误，要在数据前面加入 GIF 的数据，得到完整的图片。做题的时候也会遇到错误数据压缩包 RAR 等等，也是这样的情况都修改文件头以达到正常打开文件的目的。一般图片能正常打开后就能看到 flag。其次就是图片能够正常打开，在 010editor 中打开图片后发现最下面提示数据不匹配，这个时候可能就是图片的宽高不正确。修改宽高后就能得到完整的图片以及 flag。

![](https://img-blog.csdnimg.cn/direct/372f36389a5c47a686e10135e33b7200.png)

![](https://img-blog.csdnimg.cn/direct/a851e0224331401d9d6a33b52d3e216a.png)

解码
--

### ARCHPR

当压缩包被加密时我们就要使用这个工具，一般使用到的就是四位掩码爆破，题目可能就会有提示说是设置密码时是四位，如果没有题目提示的话我们还是先四位掩码爆破，其次是六位八位。

目前我做到的题多数都是四位掩码来爆破加密的压缩包，如果六位八位的都没爆出来，那可能这题压缩包密码的关键不是在 ARCHPR 里面解出。

![](https://img-blog.csdnimg.cn/direct/4dfaf22508504c5ebd41316aefa642ae.png)

![](https://img-blog.csdnimg.cn/direct/440614f39afb4b9384eb1d10852fa368.png)

![](https://img-blog.csdnimg.cn/direct/8b7dcb86e0374813a43d1998417da07c.png)

还有压缩包是被加密的状态，但是我们怎么也解密不出来的情况，题目提示伪加密，也就是它其实根本没有密码，所以是不可能解出来的

![](https://img-blog.csdnimg.cn/direct/315715b802d040d8b3b0fbccbf7c3c4e.png)

未加密： 文件头中的全局方式位标记为 00 00  目录中源文件的全局方式位标记为 00 00

伪加密： 文件头中的全局方式位标记为 00 00 目录中源文件的全局方式位标记为 09 00

真加密： 文件头中的全局方式位标记为 09 00 目录中源文件的全局方式位标记为 09 00

这题就是将两个 09 00 改为 00 00 就能打开压缩包

二维码
---

其实要是直接给出二维码的话还是好解决的，就是直接将二维码放到 QR 工具里面就能找到 flag（或者不能直接得到 flag，而是得到其他有效信息，这些有效信息肯定也能促进我们做题的进度），要是考到这样的题目的话就是怎么得到二维码可能才是考点。至于怎么获得二维码就可能用到前面的知识点或者其他更深层次的知识，这个可以经过多刷题自己总结。这里就只说要是遇到二维码就直接使用 QR 扫描得到有效信息。

![](https://img-blog.csdnimg.cn/direct/5e036d53581c4bd99c4dd5ae851340cf.png)

密码
--

做杂项的时候有能多题都是和密码学相连在一起的，关键信息在密码中提示。

这些密码就是做题得做的多，看到密文就能知道对应到什么密文，然后在百度在线解密。

当不知道看到的密码是什么密文加密，首先就是看题目的主题是什么，在百度上面查找看看有没有类似主题的密文，然后解密看看是不是我们要的密钥。

![](https://img-blog.csdnimg.cn/direct/416623a2080b42c9b258c9527905df65.png)

![](https://img-blog.csdnimg.cn/direct/658b5366575b49cebc137bd9b65a972a.png)

![](https://img-blog.csdnimg.cn/direct/eb25aca718034feaa9279429071e9506.png)

音频题，就是给一段视频或者音频，相关的密文就是摩斯码

使用 Audacity 工具，得到摩斯码

![](https://img-blog.csdnimg.cn/direct/e30eac6d56d14628bc6d13ad67f97225.png)

看图得摩斯码，长一点的蓝色是短线，短一点的蓝色是点，间隔就是空格，使用百度在线解码就能得到 flag

如果不想使用工具的话也能自己听声音，摩斯码是由空格短线和点三个部分组成，长滴声就是短线，短促的滴声就是点，声音的间隔就是空格。（但是这样的错误率可能会比较高）

![](https://img-blog.csdnimg.cn/direct/11becd2bb56a488490d1e57c4c0aafd7.png)

与佛论禅密文

还有很多密文就是自己刷题总结，把他们看眼熟，实在不行就是百度查看主题有没有什么相关密文。