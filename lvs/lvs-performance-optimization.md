# LVS 性能优化

LVS 全称 "Linux Virtual Server" 即 Linux 虚拟服务器，由前阿里云CTO章文嵩开发，已在国内外大型互联网公司广泛使用。LVS 主要用于 LB 负载均衡集群(load balance)。LVS 是基于 Linux 内核以模块的方式进行工作。LVS 的工作原理是，接受客户端请求，然后进行调度算法处理，将请求分发给后端服务器(Real Server)进行业务处理。LVS 本身只做流量转发，不涉及到业务处理，所以 LVS 是一种相对高效的软件负载均衡解决方案。
   
## LVS术语和概念
   
`DS` "Director Server" 也就是前端 LVS 负载均衡服务器
`RS` "Real Server" 指的是后端真实的工作服务器
`VIP` "Virtual Server IP" 对外部客户端提供服务的 IP 地址，一般配置在 LVS 服务器上
`DIP` "Director Server IP" LVS 服务器用于和后端 RS 服务器进行通信的 IP 地址
`RIP` "Real Server IP" 后端 real server 真实服务器 IP 地址
`CIP` "Client IP" 指的是客户端 IP 地址
   
## LVS工作模式
   - NAT 模式
     
   通过网络地址转换，调度器重写请求报文的目标地址，根据预设的调度算法，将请求分派给后端的真实服务器，真实服务器的响应报文通过调度器时，报文的源地址被重写，再返回给客户，完成整个负载调度过程。
   
   - TUN 模式
   采用NAT技术时，由于请求和响应报文都必须经过调度器地址重写，当客户请求越来越多时，调度器的处理能力将成为瓶颈。为了解决这个问题，调度器把请求报文通过IP隧道转发至真实服务器，而真实服务器将响应直接返回给客户，所以调度器只处理请求报文。由于一般网络服务应答比请求报文大许多，采用 TUN 技术后，集群系统的最大吞吐量可以提高10倍。
   
   - DR 模式
   DR模式下通过改写请求报文的MAC地址，将请求发送到真实服务器，而真实服务器将响应直接返回给客户。同 TUN 模式一样，DR模式可极大地提高集群系统的伸缩性。这种方法没有IP隧道的开销，对集群中的真实服务器也没有必须支持IP隧道协议的要求，但是要求调度器与真实服务器都有一块网卡连在同一物理网段上。

   - Full-NAT 模式
   LVS 的 Full-Nat 工作模式是淘宝开源的一个新的工作模式，需要编译 Linux 内核安装补丁。

## LVS调度算法
   
   - 轮叫（Round Robin）
   调度器通过"轮叫"调度算法将外部请求按顺序轮流分配到集群中的真实服务器上，它均等地对待每一台服务器，而不管服务器上实际的连接数和系统负载。
   
   - 加权轮叫（Weighted Round Robin）
   调度器通过"加权轮叫"调度算法根据真实服务器的不同处理能力来调度访问请求。这样可以保证处理能力强的服务器处理更多的访问流量。调度器可以自动问询真实服务器的负载情况，并动态地调整其权值。
   
   - 最少链接（Least Connections）
   调度器通过"最少连接"调度算法动态地将网络请求调度到已建立的链接数最少的服务器上。如果集群系统的真实服务器具有相近的系统性能，采用"最小连接"调度算法可以较好地均衡负载。
   
   - 加权最少链接（Weighted Least Connections）
   在集群系统中的服务器性能差异较大的情况下，调度器采用"加权最少链接"调度算法优化负载均衡性能，具有较高权值的服务器将承受较大比例的活动连接负载。调度器可以自动问询真实服务器的负载情况，并动态地调整其权值。
   
   - 基于局部性的最少链接（Locality-Based Least Connections）
   "基于局部性的最少链接"调度算法是针对目标IP地址的负载均衡，目前主要用于Cache集群系统。该算法根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器；若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则用"最少链接"的原则选出一个可用的服务 器，将请求发送到该服务器。
   
   - 带复制的基于局部性最少链接（Locality-Based Least Connections with Replication）
   "带复制的基于局部性最少链接"调度算法也是针对目标IP地址的负载均衡，目前主要用于Cache集群系统。它与LBLC算法的不同之处是它要维护从一个目标IP地址到一组服务器的映射，而LBLC算法维护从一个目标IP地址到一台服务器的映射。该算法根据请求的目标IP地址找出该目标IP地址对应的服务器组，按"最小连接"原则从服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器，若服务器超载；则按"最小连接"原则从这个集群中选出一 台服务器，将该服务器加入到服务器组中，将请求发送到该服务器。同时，当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的 程度。
   
   - 目标地址散列（Destination Hashing）
     
   "目标地址散列"调度算法根据请求的目标IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空。
   
   - 源地址散列（Source Hashing）
     
   "源地址散列"调度算法根据请求的源IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空。

   - 最短期望延迟(Shortest Expected Delay)
   基于wlc算法，简单算法：（active + 1) * 256 / weight 也就是 "(活动连接数 + 1) * 256 / 权重"

   - 永不排队 (Never Queue）
   改进版的 sed 算法，无需队列，如果有台 realserver 的连接数等于 0 就直接分配过去，不需要再进行 sed 运算。

## LVS命令行工具

加载 ip_vs 内核模块
   
```
$ modprobe ip_vs
```
   
安装 LVS 的管理工具 ipvsadm
   
```
$ yum install -y ipvsadm
```
   
创建 LVS 虚拟服务器，调度算法为 =rr= 轮叫算法
   
```
$ ipvsadm -A -t 112.2.21.88:80 -s rr
```
   
添加后端 real server 服务器节点，使用 DR 模式
   
```
$ ipvsadm -a -t 112.2.21.88:80 -r 10.10.24.21:80 -g
```
   
删除一个 LVS 后端服务器节点
   
```
$ ipvsadm -d -t 112.2.21.88:80 -r 10.10.24.21:80
```
   
清空所有 LVS 虚拟服务器和后端服务器节点
   
```
$ ipvsadm -C
```
   
查看 LVS 状态
   
```
$ ipvsadm --list --stats
```
   
**LVS 参数选项**
   
```
-A 在 LVS 中添加一个新的虚拟服务器
-a 在虚拟服务器中添加一个后端真实服务器节点
-C 清除 LVS 中所有服务器
-d 删除一个后端真实服务器
-D 删除一个虚拟服务器
-e 修改后端真实服务器参数
-E 修改虚拟服务器参数
-l 显示 LVS 的所有列表信息
-t 表示为 tcp 服务
-u 表示为 udp 服务
-s 使用的调度算法 rr | wrr | lc | wlc | lblb | lblcr | dh | sh | sed | nq
-S 保存 LVS 服务器规则列表，默认输出到标准输出
-t 表示虚拟服务器提供tcp服务
-r 后端真实服务器地址
-R 恢复 LVS 的服务器规则列表，默认从标准输入读取
-m 指定LVS工作模式为NAT模式
-n 以数字形式显示 IP 地址和端口号
-w 后端真实服务器的权值
-g 指定LVS工作模式为直接路由器模式(也是LVS默认的模式)
-i 指定LVS的工作模式为隧道模式
-p 会话保持时间，同一个客户端被分发到同一个后端服务器的会话时间
-Z 将服务器相关计数器数据清零
--sort 对后端真实服务器进行排序输出
--rate 显示速率信息
--stats 显示统计信息
--daemon 显示同步守护进程状态
--timeout 显示 tcp tcpfin udp 的 timeout 值
--connection 显示连接统计信息
--thresholds 显示相关阀值信息
--persistent-conn 显示持久连接信息
```

## LVS相关问题与性能优化

**网卡 LRO/GRO 功能**

现在大多数网卡都具有 LRO/GRO 功能，也就是将多个小数据包合并成一个大数据包，当数据包大小超过 MTU 1500bytes 的时候，经过内核协议栈，LVS 就会直接丢弃。所以，有的时候我们用 LVS 来传输大文件，就很容易出现丢包，传输速度慢的现象，这个时候就需要关闭网卡的 LRO/GRO 功能。

首先需要安装 ethtool 工具

```
$ yum install -y ethtool
```

查看网卡 LRO/GRO 功能状态

```
$ ethtool -k eth0
```

关闭网卡 LRO/GRO 功能

```
$ ethtool -k eth0 lro off
$ ethtool -k eth0 gro off
```

**增大连接 Hash 表大小**

默认情况下 LVS 的最大并发数取决于它的连接 Hash 表的大小，默认为 4096

```
$ ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
-> RemoteAddress:Port           Forward Weight ActiveConn InActConn
```

通过修改 LVS 模块配置文件 /etc/modprobe.d/ip_vs.conf 来增大 LVS 的并发连接数

```
$ emacs /etc/modprobe.d/ip_vs.conf
options ip_vs conn_tab_bits=20
```

重启服务器后查看 LVS Hash 表大小

```
$ ipvsadm -ln
IP Virtual Server version 1.2.1 (size=1048576)
Prot LocalAddress:Port Scheduler Flags
-> RemoteAddress:Port           Forward Weight ActiveConn InActConn
```

**禁用ARP增大backlog并发数**

修改内核配置文件 /etc/sysctl.conf

```
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.core.netdev_max_backlog = 102400
```

**tcp tcpfin udp 超时时间**

在一些长连接应用场景中这 3 个参数非常重要，并且 tcp 参数必须大于 tcpfin，单位是秒。

**理解 persistent 超时时间**

LVS 的 persistent 超时时间指的是，在这个时间内相同 IP 地址的客户端会分发到同一台后端服务器，当这个时间变成 0 时，如果还有客户端连接，它会重新开始计时。如果没有客户端连接就会自动删除，相同 IP 地址客户端下次就会被分发到另外一台后端服务器节点，一般这个时间值大于 LVS 的 tcp + tcpfin 之和即可，但不要太大。

**多队列网卡中断优化**

使用 lspci 命令查看网卡是否支持多队列

```
$ lspci -vvv
```

查看 "Ethernet controller" 以太网控制器中，如果有 MSI-X 并且 Enable+ Count > 1 就表示支持多队列

```
00:04.0 Ethernet controller: Red Hat, Inc Virtio network device
        Subsystem: Red Hat, Inc Device 0001
        Physical Slot: 4
        Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR+ FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0
        Interrupt: pin A routed to IRQ 11
        Region 0: I/O ports at c080 [size=32]
        Region 1: Memory at febd2000 (32-bit, non-prefetchable) [size=4K]
        Expansion ROM at feb80000 [disabled] [size=256K]
        Capabilities: [40] MSI-X: Enable+ Count=3 Masked-
                Vector table: BAR=1 offset=00000000
                PBA: BAR=1 offset=00000800
        Kernel driver in use: virtio-pci
        Kernel modules: virtio_pci
```

**查看所有网卡中断号**

```
$ cat /proc/interrupts
IRQ        CPU0      CPU1       CPU2       CPU3
122:       1709      25131    3548386     117109        PCI-MSI-X  eth0-0
130:        441    2167083  205623005    7779272        PCI-MSI-X  eth0-1
138:        418    4803724  109660398    7188985        PCI-MSI-X  eth0-2
146:        389    5473623   97942351    6893766        PCI-MSI-X  eth0-3
154:        397    4999666   83224034    6996673        PCI-MSI-X  eth0-4
162:        399    4542680   84930436    8034803        PCI-MSI-X  eth0-5
170:        406    4429835  132193602    8443930        PCI-MSI-X  eth0-6
178:        449    2881815  172147095    5454282        PCI-MSI-X  eth0-7
```

输出结果中的第 1 列表示中断号

**手动设置将中断分配到不同的CPU核心上**

例如将 eth0 网卡的 0 号队列，中断号为 122 绑定到第 1 个 CPU 核心上

```
$ echo 1 > /proc/irq/122/smp_affinity
```

例如将 eth0 网卡的 1 号队列，中断号为 130 绑定到第 2 个 CPU 核心上

```
$ echo 2 > /proc/irq/130/smp_affinity
```

例如将 eth0 网卡的 2 号队列，中断号为 138 绑定到第 3 个 CPU 核心上

```
$ echo 4 > /proc/irq/138/smp_affinity
```

smp_affinity 文件中使用十进制数表示 CPU 核心位

```
00000001  1     第 1 个 CPU
00000010  2     第 2 个 CPU
00000100  4     第 3 个 CPU
00001000  8     第 4 个 CPU
00010000  16    第 5 个 CPU
00100000  32    第 6 个 CUP
01000000  64    第 7 个 CPU
10000000  128   第 8 个 CPU
```

注意上例中的 eth0 网卡只能同时绑定到一个 CPU 核心上，不能同时平均分配到所有 CPU 上。假如有两块网卡 eth0 和 eth1，我们就可将 eth0 绑定到第 1 个 CPU 上，eth1 绑定到第 2 个 CPU 上。在手动绑定 CPU 中断前，必须停止 irqbalance 服务，否则手动配置会被覆盖。
