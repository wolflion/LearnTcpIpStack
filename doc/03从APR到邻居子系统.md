## 3.1、rfc826

### 0、

+ 代码目录
  + include/linux/if_arp.h，ARP报文结构，函数原型
  + net/ipv4/arp.c，ARP协议的实现

### 3.1、rfc826到ARP.c

#### 3.1.1、rfc826格式

##### Definitions：

+ TYPE field of the Ethernet packet header
  +  `ether_type$XEROX_PUP`, 
  + `ether_type$DOD_INTERNET`, 
  + `ether_type$CHAOS`,
  + *还有几个没懂呢*
+ 对应的是 中文标注的**硬件类型**

##### Packet format:

+ Ethernet transmission layer (not necessarily accessible to
  the user):  *好好读，其实人家写得很清楚*，【第一时间怀疑下自己没看懂】，**以太网首部**
  + Ethernet address of destination，**以太网目的地址**
  + Ethernet address of sender，**以太网源地址**
  + 16.bit: Protocol type =ether_type$ADDRESS_RESOLUTION，**帧类型**，（*感觉对不上*）

+ Ethernet packet data:
  + 16.bit: (ar$hrd) Hardware address space (e.g., Ethernet,Packet Radio Net.)
  + `16.bit: (ar$pro) Protocol address space. For Ethernet
    hardware, this is from the set of type
    fields ether_typ$<protocol>.`
  + 8.bit: (ar$hln) byte length of each hardware address
  + 8.bit: (ar$pln) byte length of each protocol address
  + `16.bit: (ar$op) opcode (ares_op$REQUEST | ares_op$REPLY)`，**这个地方提到的操作码**
  + `nbytes: (ar$sha) Hardware address of sender of this packet, n from the ar$hln field.`
  + `mbytes: (ar$spa) Protocol address of sender of this packet, m from the ar$pln field.`
  + nbytes: (ar$tha) Hardware address of target of thispacket (if known).
  + mbytes: (ar$tpa) Protocol address of target.

##### Packet Generation:

+ *有点长，没看懂*

##### Packet Reception:

+ *也没看*

##### Why is it done this way??

##### Network monitoring and debugging:



#### 3.1.2、代码解析

#### 参考

+ rfc826，https://www.rfc-editor.org/rfc/pdfrfc/rfc826.txt.pdf，ipv4相关
+ [详解RFC 826文档](https://zhuanlan.zhihu.com/p/427573801)，zhihu上的
+ 《Linux内核源码剖析-TCP/IP实现》chap18、ARP：地址解析协议  （490/560）
+ https://xdksx.github.io/2021/05/22/linux-neighbor/

### 3.2、邻居发现协议

#### 参考

+ rfc4861，*邻居发现协议*，共93页
+ rfc2861，*最新版的更新一点*
+ csdn，qq_53111905，爱好学习的青年人，linux内核邻接子系统（arp协议）的工作原理
+ csdn.net，weiqian1991，网络协议栈的源码部析