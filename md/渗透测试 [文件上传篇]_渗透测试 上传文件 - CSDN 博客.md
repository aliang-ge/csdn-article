> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/npc88888/article/details/136400867)

### 一. 绕过 js 检测方法

> 按 F12 使用网页审计元素，把校验的上传文件后缀名文件删除，即可上传。

![](https://img-blog.csdnimg.cn/direct/3b4d0d006a074eb98e87d2f7a9354d28.png)

> 把恶意文件改成 js 允许上传的文件后缀，如 jpg、gif、png 等，再通过[抓包](https://so.csdn.net/so/search?q=%E6%8A%93%E5%8C%85&spm=1001.2101.3001.7020) 工具抓取 post 的数据包，把后缀名改成可执行的脚本后缀如 php 、asp、jsp、 net 等。即可绕过上传。

把 s.php 改变后缀名为 s.jpg

```
<?php phpinfo();eval($_POST['cmd']);?>
 
#一句话木马
```

点击上传文件

![](https://img-blog.csdnimg.cn/direct/a84c32823afe42379de3ef961444b6d4.png)

抓包，把文件名后缀改为 php

![](https://img-blog.csdnimg.cn/direct/d7148015d90b4bf8bc6bbde46980eab2.png)

上传成功

![](https://img-blog.csdnimg.cn/direct/ce982eed15b844c69ec58a31d4909e60.png)

### 二. 绕过 contnet-type 检测上传

> 服务端是通过 content-type 判断类型， content-type 在客户端可被修改。

![](https://img-blog.csdnimg.cn/direct/ede30e7e638240f599cc8f1b2352f8ec.png)

![](https://img-blog.csdnimg.cn/direct/4edea3209f12448a8d329f6be443d840.png)

### 三. 绕过黑名单检测

> 上传图片时，如果提示不允许 php、asp 这种信息提示，可判断为黑名单限制， 上传黑名单以外的后缀名即可。 在 iis 里 asp 禁止上传了，可以上传 asa cer cdx 这些后缀，如在网站里允许. net 执行 可以上传 ashx 代替 aspx。如果网站可以执行这些脚本，通过上传后门即可 获取 webshell。 在不同的中间件中有特殊的情况，如果在 apache 可以开启 application/x-httpd-php 在 AddType application/x-httpd-php .php .phtml .php3 后缀名为 phtml 、php3 均被解析成 php 有的 apache 版本默认就会开启。 上传目标中间件可支持的环境的语言脚本即可，如. phtml、php3。
> 
> 就是把不同的文件后缀放上去试试，有的成功上传后还解析不了。我个人觉得没什么用 

#####  1.htaccess 重写解析绕过上传

```
<FilesMatch "jpg">
SetHandler application/x-httpd-php
</FilesMatch>
#意思是把jpg文件当php文件解析
```

过程就是把这两个文件放进去

![](https://img-blog.csdnimg.cn/direct/24681c4e8d594c0c82b9eed076ba5d80.png)然后访问 s.jpg

![](https://img-blog.csdnimg.cn/direct/c8a52b30d191494eb9d488833504e9c7.png)

##### 2. 大小写绕过

> 源代码中没有过滤 PHP,phP 为后缀的文件。所以直接就注入了

![](https://img-blog.csdnimg.cn/direct/15c26049ddc345248428c407c25ea039.png)

##### 3. 空格绕过上传

> 例如这题，只用`($_FILES['upload_file']['name']);`获取了上传文件的名称，却没有拿 trim 包着删除文件名中可能存在的空白字符。这就导致了空格绕过的安全漏洞

![](https://img-blog.csdnimg.cn/direct/60b963ef74df4108a6612340803d50c0.png)

有些浏览器是不支持上传带空格的文件的，所以拿 burp suite 抓包，加空格后再上传。

![](https://img-blog.csdnimg.cn/direct/7772be8e03234920a26509db759612cc.png)

##### 4. 利用 windows 系统特征绕过上传

> 在 windows 中文件后缀名. 系统会自动忽略. 所以 shell.php. 像 shell.php 的效果一 样。所以可以在文件名后面机上. 绕过。

 但是这题利用的不是 windows 系统特性

![](https://img-blog.csdnimg.cn/direct/98fb45d932f64271a63e9fed4407f9f6.png)

![](https://img-blog.csdnimg.cn/direct/66e396abb30d4f0e83a4c3dc125e64f8.png)

##### 5.NTFS 交换数据流::$DATA 攻击绕过上

> 在一般情况下，大多数应用程序会将 `::$DATA` 部分视为文件名的一部分，而并非数据流。所以，对于大多数应用程序来说，`s.php::$DATA` 与 `s.php` 是等效的，都会被视为同一个文件名。

burpsuite 抓包，修改后缀名为 php::$DATA

![](https://img-blog.csdnimg.cn/direct/92f2b7d38947434faf1c09f86a265373.png)上传成功 ![](https://img-blog.csdnimg.cn/direct/aff51c6f66fd49f696133e759c5b1083.png)

##### 6. 利用 windows 环境的叠加特征绕过上传攻击

> 首先抓包上传 a.php:.php 上传会在目录里生成 a.php 空白文件，接着再次提交把 a.php 改成 a.>>
> 
> 文件名中，`s.php:.php` 这样的命名实际上是一个非常特殊且不常见的情况。在这种情况下，`s.php:` 是文件名的基本部分，而 `.php` 可能被视为文件的扩展名。

 ![](https://img-blog.csdnimg.cn/direct/77dc430c694b4969a27f4bdac64560d6.png)

上传的文件名为 s.php 但是是为空的

![](https://img-blog.csdnimg.cn/direct/8de597acc3c24e4eadaae09c3ddb0ba3.png)

重新输入 s.<<< 就有内容了

> 在 windwos 中如果上传文件名 moonsec.php:.jpg 的时候，会在目录下生产空白的 文件名 moonsec.php 再利用 php 和 windows 环境的叠加属性， 以下符号在正则匹配时相等
> 
> 双引号 " 等于 点号.
> 
> 大于符号 > 等于 问号?
> 
> 小于符号 < 等于 星号 *
> 
> 所以 s.<<<或 s.>>> 就等价于 s.*** 或 s.??? 匹配相应的文件然后写入内容

![](https://img-blog.csdnimg.cn/direct/fb0692abb74245d8ae3b13879c11e430.png)

##### 7. 文件上传双写绕过漏洞分析

代码分析

![](https://img-blog.csdnimg.cn/direct/a3d8967e2f3f4d6db58e08b5c36161bf.png)

 ![](https://img-blog.csdnimg.cn/direct/fd264078f7b748deb16656956f983982.png)

![](https://img-blog.csdnimg.cn/direct/8586f948c5e84918bb91b923ca9e42e8.png)

###  四. 文件上传参数目录可控攻击

##### 1. 目录可控 get 绕过上传

> 传入一个文件然后文件名和路径由 $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext; 定义。

 ![](https://img-blog.csdnimg.cn/direct/c7c97f6514e34080aaf702ff69af481e.png)

具体就是先传入 s.jpg 绕过白名单过滤，在 get 那里传入文件名 %00 是截断下面文件名的

![](https://img-blog.csdnimg.cn/direct/08c627679d7e46949c06efbce02fdd62.png)

![](https://img-blog.csdnimg.cn/direct/ec4015ca2da441c486f24d0ac52aec0d.png)

##### 2. 目录可控 POST 绕过上传

![](https://img-blog.csdnimg.cn/direct/acbd8e1d93d741f49546953b07c9126c.png) ![](https://img-blog.csdnimg.cn/direct/6d26b89ecdca4a10ac5026d0ae075052.png)

![](https://img-blog.csdnimg.cn/direct/bdcd94f68a0b48b490beef0cc21bfb1c.png)

### 五.. 文件头检测绕过上传

> 有的文件上传，上传时候会检测头文件，不同的文件，头文件也不尽相同。常见 的文件上传图片头检测 它检测图片是两个字节的长度，如果不是图片的格式， 会禁止上传。
> 
> 常见的文件头
> 
>  JPEG (jpg)，文件头：FFD8FF
> 
>  PNG (png)，文件头：89504E47
> 
>  GIF (gif)，文件头：47494638
> 
>  TIFF (tif)，文件头：49492A00
> 
>  Windows Bitmap (bmp)，文件头：424D 

##### 1. 制作图片一句话

使用  copy s.jpg/b + s.php/a shell.jpg 将 php 文件附加再 jpg 图片上，直接上传即可。 

代码分析

![](https://img-blog.csdnimg.cn/direct/f4ab403506864c95b023e529f5f79bdb.png)

放入图片马

![](https://img-blog.csdnimg.cn/direct/a1ed38fc330e4be7aa183544ac6e356d.png)

执行

![](https://img-blog.csdnimg.cn/direct/5ae49b21b751489eb2060699f1131342.png)

##### 2.getimagesize 是获取图片的大小，如果头文件不是图片会报错直接可以用图片马 绕过检测。、

![](https://img-blog.csdnimg.cn/direct/2a00f76d188040eb9d34ee1fbe57f3c3.png)

##### 3. 图片二次渲染分析代码

![](https://img-blog.csdnimg.cn/direct/6bee796cc7a842a283f3fd5be0543547.png)

> 只允许上图片二次渲染分析代码 只传 JPG PNG gif 在源码中使用 imagecreatefromgif 函数对图片进行二次 生成。生成的图片保存在，upload 目录下。

渗透步骤

双击点开

![](https://img-blog.csdnimg.cn/direct/47f6488025ef4fcabd22aeeb903164ef.png)

 查看上传到网站上渲染过的图片和原本的图片

![](https://img-blog.csdnimg.cn/direct/9c5a102dbd5445f4938ab6b82762e443.png)

![](https://img-blog.csdnimg.cn/direct/a96539efc6674ac7807cba65c70be524.png)

##### 4. 文件名控代码分析

![](https://img-blog.csdnimg.cn/direct/a0ae61077568483584aaff39b9bb79ef.png)

> `pathinfo($file_name, PATHINFO_EXTENSION)`：`pathinfo()` 函数用于返回文件路径的信息。通过设置第二个参数 `PATHINFO_EXTENSION`，可以获取文件的扩展名部分

 把 post 请求改为 s.php%001.jpg，`pathinfo()`会把后面的 1.jpg 过滤掉![](https://img-blog.csdnimg.cn/direct/649bda8d00934eeba18a0657f0b5dca1.png)

然后把 %00 编码

![](https://img-blog.csdnimg.cn/direct/fccfc0679f8d4267996470b5681bfeb1.png)

### 六. 数组绕过代码分析

![](https://img-blog.csdnimg.cn/direct/b051f24d5c3948299809322a0a2e61ff.png)

![](https://img-blog.csdnimg.cn/direct/2a9110ca21144f1c9962d9c69fb98e51.png)