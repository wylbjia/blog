# tcpdump 工具使用教程

tcpdump 是一个实用的网络数据包分析工具，能够截获操作系统中的各种网络数据包。

## tcpdump 参数选项列表

```
-A
```

使用 ASCII 本文可读方式打印每个数据包信息，通常用于获取 http 等协议的数据，不可与 `-X` 参数同时使用

```
-b
```

```
-B buffer_size, --buffer-size=buffer_size
```

设置操作系统用于 tcpdump 数据包捕获缓冲区的大小，单位为 KiB (1024 bytes)
```
-c count
```

当接收到这个参数指定的数据包数量后 tcpdump 自动退出

```
-C file_size
```


```
-d
```

将获取到的匹配数据包代码以可读的形式输出到屏幕，然后自动退出

```
-dd
```

将获取到的匹配数据包代码以 C 语言程序片段形式输出到屏幕，然后自动退出

```
-ddd
```

将获取到的匹配数据包代码以十进制的形式输出到屏幕，然后自动退出

```
-D, --list-interfaces
```

输出系统中所有可用的网络接口列表

```
-e
```

在每个数据包输出中打印链路层头信息，例如 MAC 层协议和以太网层信息

```
-E
```

使用 des-cbc, 3des-cbc, blowfish-cbc, rc3-cbc, cast128-cbc 等算法解密 IPsec 的 ESP 数据包

```
-f
```

```
-F file
```

从一个文件中读取过滤表达式，并且会忽略命令行参数中的过滤表达式

```
-G rotate seconds
```

```
-h, --help
```

输出 tcpdump 的帮助信息

```
--version
```

输出 tcpdump 和 libpcap 的相关版本信息

```
-H
```

尝试检测 802.11s 协议草案中规定的头信息

```
-i interface, --interface=interface
```

设置用于 tcpdump 监听捕获数据包的网络接口

```
-I, --monitor-mode
```

让接口工作在监听模式下，仅支持 IEEE 802.11 WiFi 的接口， 

```
--immediate-mode
```

```
-j tstamp_type, --time-stamp-type=tstamp_type
```

```
-J, --list-time-stamp-types
```

```
--time-stamp-precision=tstamp_precision
```

```
-K, --dont-verify-checksums
```

不验证 IP, TCP 或者 UDP 的数据包校验和

```
-l
```

```
-L, --list-data-link-types
```

```
-m module
```

```
-M secret
```

```
-n
```

不将主机地址解析成域名

```
-nn
```

不将主机地址解析成域名，并且不会将端口解析成服务

```
-N
```

```
-#, --number
```

在每一个数据包行前面打印一个可选的包编号

```
-O, --no-optimize
```

```
-p, --no-promiscuous-mode
```

```
-Q direction, --direction=direction
```

```
-q
```

快速输出，仅列输出精简的协议信息

```
-r file
```

```
-S, --absolute-tcp-sequence-numbers
```

打印出绝对的 TCP 序列号，tcpdump 默认输出的是相对序列号

```
-s snaplen,  --snapshot-length=snaplen
```

设置 tcpdump 数据包大小，默认为 262144 字节，当超过这个大小数据会被截断，设置为 0 表示获取所有数据

```
-T type
```

```
-t
```

不在每行数据包前面打印时间戳

```
-tt
```

在每一行数据包前打印时间戳，也就是从 1970 年开始至今的秒数

```
-ttt
```

在每一行上打印出当前行数据包与上一行数据之间的时间差值

```
-tttt
```

在每一行上打印小时、分钟、秒格式的时间戳

```
-ttttt
```

在每一行上打印出当前行数据包与第一行数据之间的时间差值

```
-u
```

打印未加密的 NFS 处理数据包

```
-U, --packet-buffered
```

```
-v
```

输出每一个数据包的基本详细信息

```
-vv
```

输出每一个数据包的更多详细信息

```
-vvv
```

输出每一个数据包的更加详细的信息

```
-V file
```

```
-w file
```

将 tcpdump 的原始数据包写入到一个文件中，而不是输出到屏幕

```
-W
```

```
-x
```

```
-xx
```

```
-X
```

```
-XX
```

```
-y datalinktype, --linktype=datalinktype
```

```
-z postrotate-command
```

```
-Z user, --relinquish-privileges=user
```

```
expression
```

这里是一个些数据包匹配规则表达式

## tcpdump 匹配规则表达式



