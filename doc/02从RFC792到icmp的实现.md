## 2.1、从rfc792文档到代码的注释

### 0、引言

+ 代码
  + net/ipv4/icmp.c
    + inet_init()里会加载`if (inet_add_protocol(&icmp_protocol, IPPROTO_ICMP) < 0)`
  + net/ipv4/af_inet.c
    + af是指**地址族**（address family），里面有多个，含有inet

+ ICMP报文会分为**查询**和**差错**报文2种
  + *哪种情况下，会用查询报文，哪种情况用差错呢？*
  + 代码中，error为1是差错，error为0是查询

### 2.1.1、对比rfc792阅读icmp.c

#### rfc792

+ ICMP报头
  + *属于几层？*IP层，*它需不需要在前面加头？*
  + 前3个是Type、Code、checksum
    + 比如14 for timestamp reply message，时间戳应答
+ Destination Unreachable Message
  + 分几种情况，port、net、host等等

#### icmp.c

+ 协议的初始化，`icmp_init()`
  + **因为ICMP无法被编译成模块，所以没有module_init这种**
  + for_each_possible_cpu(i)，*不是很明白*
  + *封装了一个函数inet_ctl_sock_create，去掉了per_cpu参数*
  + *整体改动还是比较大的*
+ 怎么关联
  + `icmp_pointers[NR_ICMP_TYPES + 1]`里面的handler，对应各种icmp_X函数
  + *这个机制是什么*？
+ 传输ICMP消息
  + icmp_recv()
  + icmp_reply()
+ 接收ICMP消息
  + icmp_rcv()
    + *樊东东书中的图14-2不错，画一下，放到blog中*
    + Parse the ICMP message ，这部分，*其实还是根据文档来的*

### 2.1.2、工具与代码

+ ping或traceroute

### 参考

+ rfc792，wiki上，https://zh.wikipedia.org/wiki/%E4%BA%92%E8%81%94%E7%BD%91%E6%8E%A7%E5%88%B6%E6%B6%88%E6%81%AF%E5%8D%8F%E8%AE%AE
+ rfc792，官网上，https://www.rfc-editor.org/pdfrfc/rfc792.txt.pdf
+ 《深入理解Linux网络技术内幕》chap25
+ 《追踪Linux TCP／IP代码运行：基于2.6内核》chap13
+ 《Linux内核源码剖析-TCP/IP实现》chap14
+ 《TCP/IP详解卷1：协议》chap6
+ 26ICMP解析，https://sunyunqiang.com/blog/icmp_protocol/，*好像打不开*