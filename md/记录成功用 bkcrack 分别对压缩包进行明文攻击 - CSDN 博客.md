> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Rick66Ashley/article/details/130015948)

这次因为是学习，所以我就用了 [buuctf](https://so.csdn.net/so/search?q=buuctf&spm=1001.2101.3001.7020) 上的题

就是 ACTF 明文攻击那题...

终于行了！终于行了！yeah！

我主要集中说说明文攻击那部分。

首先，拿到附件，会有一个压缩包 res.zip 和一个图片。

用 [binwalk](https://so.csdn.net/so/search?q=binwalk&spm=1001.2101.3001.7020) 分析，有压缩包，但是提不出来?

010 一看，文件头有问题，手动修复，然后分离放到新的是文件里面。打开，发现分离压缩包有 flag.txt 但是不是答案。再看 res.zip，用 010 一看，真加密；且里面也有 flag.txt 还有一个 secret.txt 说明是明文攻击，且 flag 在 secret 里面。

ok! 开始明文！

1. 打开 bkcrack 所在文件，在框里 cmd

2. 输入 bkcrack -C res.zip -c flag.txt -P ohh.zip -p flag.txt，其中 - C 表示密文（[cipher](https://so.csdn.net/so/search?q=cipher&spm=1001.2101.3001.7020)），-p 为明文（plaintext），明文和密文中明文的部分对应，这么说是因为上午试过了把 secret.txt 或者整个压缩包作为密文，然后都找不出 key！！！TAT

然后 - C 应该是指外层文件，-c 应该是内层文件

3. 开 run！

![](https://img-blog.csdnimg.cn/4bcfeabfe5ad44ffb7738737bcee6780.png)

 4.OK！！GET 到 key 了

5. 下一步用 bkcrack -C res.zip -c flag.txt -k key -d new.zip

想弄成解了密的 zip 或者单个解密的 txt，都不行！！！

压缩包打不开 QAQ，保存成 txt 也是乱码勒 这里 - d 表示已解密文件

![](https://img-blog.csdnimg.cn/9d20acb4f5cc4883ad996e6c7e830285.png)

 6. 怎么办呢？后来用了 - U 选项，就是把利用得到的算法密钥把压缩包密码改了

bkcrack -C res.zip -k key -U new.zip good

这条指令，-U 表示更改密码 前面是新压缩包的名，后面是你设置的密码，欧克，flag 拿到了！![](https://img-blog.csdnimg.cn/4cb7d7087c594917af749acab300e3f9.png)

7. 统计一下学到的东西：

1 明文攻击是在一部分文件内容已知的情况下才可以使用  
2bkcrack -L xxx.zip 可以查看压缩包加密算法，只有加密算法为 zipcrypto 才可以使用明文攻击

AES 不行；然后就是明文至少知道十二个字节的内容，其中八个连续才行

3.bkcrack 中：

-c 密文 -p 明文 大写的是压缩包

-U 更改密码 -d 解密文件 -k 算法密钥

-r(恢复密码)  int（表示密码的最大位数） ？p（后面的是字符类型，p 是可打印字符，d 是数字，-l 小写字母，-u 大写字母）-o 表示明文相对于密文开头的偏移量（这个我现在还不会）

-c 组合 - p 拿到 key，-c 组合 - k 和 - d 拿到解密文件，-c 组合 - k 和 - u 是改压缩包密码

好，就这样了

第一篇 blog