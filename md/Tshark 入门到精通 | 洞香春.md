> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [wdd.js.org](https://wdd.js.org/network/tshark/)

> 1. 使用摘要 一个命令的使用摘要非常重要，摘要里包含了这个工具最常用的用法。

一个命令的使用摘要非常重要，摘要里包含了这个工具最常用的用法。

要注意的是，如果要用过滤器，一定要放到最后。

```
tshark [ -i <capture interface>|- ] [ -f <capture filter> ] [ -2 ] [ -r <infile> ] [ -w <outfile>|- ] [ options ] [ <filter> ]

tshark -G [ <report type> ] [ --elastic-mapping-filter <protocols> ]
```

一般情况下，我们可能会在服务端用 tcpdump 抓包，然后把包拿下来，用 wireshark 分析。那么我们为什么要学习 tshark 呢？

相比于 wireshark， tshark 有以下的优点

1.  速度飞快：wireshark 在加载包的时候，tshark 可能已经给出了结果。
2.  更稳定：wireshark 在处理包的时候，常常容易崩溃
3.  更适合做文本处理：tshark 的输出是文本，这个文本很容易被 awk, sort, uniq 等等命令处理

但是我不建议上来就学习，更建议在熟悉 wireshark 之后，再去进一步学习 tshark

3.1 基本场景
--------

用 wireshark 最基本的场景的把 pcap 文件拖动到 wireshark 中，然后可能加入一些过滤条件。

```
tshark -r demo.pcap
tshark -r demo.pcap -c 1 # 只读一个包就停止
```

输出的列分别为：序号，相对时间，绝对时间，源 ip, 源端口，目标 ip, 目标端口

3.2 按照表格输出
----------

```
tshark -r demo.pcap -T tabs
```

3.3 按照指定的列输出
------------

例如，抓的的 sip 的包，我们只想输出 sip 的 user-agent 字段。

```
tshark -r demo.pcap -Tfields -e sip.User-Agent sip and sip.Method==REGISTER
```

按照上面的输出，我们可以用简单的 sort 和 seq 就可以把所有的设备类型打印出来。

3.4 过滤之后写入文件
------------

比如一个很大的 pcap 文件，我们可以用 tshark 过滤之后，写入一个新的文件。

例如下面的，我们使用过滤器 sip and sip.Method==REGISTER, 然后把过滤后的包写入到 register.pcap

● -Y “sip and frame.cap_len> 1300” 查看比较大的 SIP 包 tshark -r demo.pcap -w register.pcap sip and sip.Method==REGISTER

3.4 统计分析
--------

tshark 支持统计分析，例如统计 rtp 丢包率。

```
tshark -r demo.pcap -qn -z rtp,streams
```

-z 参数是用来各种统计分析的，具体支持的统计类型，可以用

```
tshark -z help
➜  Desktop tshark -z help
     afp,srt
     ancp,tree
     ansi_a,bsmap
     ansi_a,dtap
     ansi_map
     asap,stat
     bacapp_instanceid,tree
     bacapp_ip,tree
     bacapp_objectid,tree
     bacapp_service,tree
     calcappprotocol,stat
     camel,counter
     camel,srt
     collectd,tree
     componentstatusprotocol,stat
     conv,bluetooth
     conv,dccp
     conv,eth
     conv,fc
```

*   [https://www.wireshark.org/docs/man-pages/tshark.html](https://www.wireshark.org/docs/man-pages/tshark.html)