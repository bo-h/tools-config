---
title: Tcpdump
date: 2017-12-23 18:19:14
tags: [Linux,Tool,Network]
toc: true
comment: true
---


# 概要

​	tcpdump命令是一款sniffer工具，根据使用者的定义对网络上的数据包进行截获，它可以打印所有经过网络接口的数据包的头信息，也可以使用-w将数据包保存到文件中，供以后分析。它支持针对网络层，协议，主机，ip和端口的过滤，并提供and， or， not等逻辑语句帮助过滤。

# 命令

>tcpdump  (选项)

#选项

> 

# 实例

### 直接启动tcpcump默认将监视第一个网络接口上所有流过的数据包

> tcpdump

### 监视指定网络接口数据包

> tcpdump -i eth1

如果不指定网卡，默认tcpdump只会监视第一个网络接口，一般是eth0

### 监视制定主机的数据包

打印所有进入或者离开localhost的数据包

> tcpdump host localhost

也可以指定127.0.0.1

> tcpdump host 127.0.0.1

打印主机localhost与hot或者是ace之间通信数据包

> tcpdump host localhost and \( hot or ace \) 

截取主机192.168.1.102和192.168.1.109的通信

> tcpdump host 192.168.1.102 and 192.168.1.109

####逻辑and, or, not, !

打印ace与其他主机之间通信的IP数据包，但是不包括hot

> tcpdump ip host ace and  not hot

截取主机192.168.1.102除了和主机192.168.1.109之外的所有通信IP包

> tcpdump ip host 192.168.1.102 and !192.168.1.109

截取主机hostname发送的所有数据

> tcpdump -i eth0 src host hostname

监视所有送到主机hostname的数据包

> tcpdump -i eth0 dst host hostname

###监视主机和端口的数据包

> tcpdump tcp port 80 host 192.168.1.102

> tcpdump udp port 23

### 监视指定网络的数据包

打印网络地址为ucb-ether的所有数据包

> tcpdump net ucb-eher

打印所有通过网关snup的ftp数据包

> tcpdump 'gateway' snup and (port ftp or ftp-data)

注意：表达式被单引号括起来了，这可以防止shell对其中的括号进行错误解析



打印所有源地址或目标地址是本地主机的IP数据包

> tcpdump ip and not net localnet

(如果本地网络通过网关连到了另一网络, 则另一网络并不能算作本地网络.(nt: 此句翻译曲折,需补充).localnet 实际使用时要真正替换成本地网络的名字)

### 监视指定协议的数据包

打印TCP会话中的的开始和结束数据包, 并且数据包的源或目的不是本地网络上的主机.(nt: localnet, 实际使用时要真正替换成本地网络的名字))

> tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'

打印所有源或目的端口是80, 网络层协议为IPv4, 并且含有数据,而不是SYN,FIN以及ACK-only等不含数据的数据包.

> tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<\<2)) - ((tcp[12]&0xf0)>>2)) != 0'

(nt: 可理解为, ip[2:2]表示整个ip数据包的长度, (ip[0]&0xf)<\<2)表示ip数据包包头的长度(ip[0]&0xf代表包中的IHL域, 而此域的单位为32bit, 要换算成字节数需要乘以4,　即左移2. (tcp[12]&0xf0)>>4 表示tcp头的长度, 此域的单位也是32bit,　换算成比特数为 ((tcp[12]&0xf0) >> 4)　<<　２,即 ((tcp[12]&0xf0)>>2).　((ip[2:2] - ((ip[0]&0xf)<\<2)) - ((tcp[12]&0xf0)>>2)) != 0　表示: 整个ip数据包的长度减去ip头的长度,再减去tcp头的长度不为0, 这就意味着, ip数据包中确实是有数据.对于ipv6版本只需考虑ipv6头中的'Payload Length' 与 'tcp头的长度'的差值, 并且其中表达方式'ip[]'需换成'ip6[]'.)



打印长度超过576字节，并且网关地址是snup的IP数据包

> tcpdump 'gateway snup and ip[2:2] > 576'

### tcpdump与wireshark

我们可以用Tcpdump + Wireshark 的完美组合实现：在 Linux 里抓包，然后在Windows 里分析包。

> tcpdump tcp -i eth1 -t -s 0 -c 100  and dst port != 22 and src net 192.168.1.0/24  -w ./target.cap

(1)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
(2)-i eth1 : 只抓经过接口eth1的包
(3)-t : 不显示时间戳
(4)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
(5)-c 100 : 只抓取100个数据包
(6)dst port ! 22 : 不抓取目标端口是22的数据包
(7)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
(8)-w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析



学习参考[Tcpdump](http://www.tcpdump.org/tcpdump_man.html)  [tcpdump.org](http://www.tcpdump.org/tcpdump_man.html)