## 缘起

+ 诉求：
  + 1、跟着书把源码过一遍（学多少算多少）**不是以书为重点，而是代码是重点**，跟书上的知识串起来，*毕竟书上讲得代码也太老了*
  + 2、跟着书，把一些流程图理解深一点（能画成什么样先什么样，能理解多少算多少）
  + 3、把过程中遇到的问题问一下AI（人家能答到什么程度，我就记一下）
+ 主要代码是
  + arp.c，if_arp.h
  + neighbour.h/c

## 内容

### 2.1、概念

+ 什么是**邻居**
  + 同一个IP局域网内的主机
  + *扩展一点，OSPF，RIP中邻居，跟它是一个意思吗？*
+ 什么是**邻居系统**，解决了什么问题
  + 知识背景：3层的IP是抽象的，2层的mac是唯一的，**但存在多种硬件的场景**（比如网卡和无线）*不一定都是mac编址*
  + 解决了**一台主机如何由一个IP地址找到对应的L2地址**
  + 邻居子系统，提供了L3与L2地址之间的映射关系，**二层首部缓存**（hh_cache），*这个作用，还不是太清楚*

### 2.2、邻居子系统的结构

+ ![自己抄的人家的结构图](https://github.com/wolflion/image/blob/main/img/%E9%82%BB%E5%B1%85%E5%AD%90%E7%B3%BB%E7%BB%9F%E7%BB%93%E6%9E%84.png)

### 2.3、ARP报文格式

### 2.4、arp.c

#### 2.4.1、static定义

##### 2.4.1.1、static定义的neigh_ops结构

+ `struct neigh_ops`定义的arp_generic_ops，arp_hh_ops，arp_direct_ops区别
  + 从代码来看，绑定的函数不一定，*那么，物理函数是啥呢*
  + arp_generic_ops，通用
  + arp_hh_ops，支持缓存硬件首部
  + arp_direct_ops，不支持ARP
    + 没有定义solicit和error_report

##### 2.4.1.2、arp_tbl，

##### 2.4.1.2、static定义的packet_type结构，arp_packet_type，定义报文类型

```c
/*
 *	Called once on startup.
 */

static struct packet_type arp_packet_type __read_mostly = {
	.type =	cpu_to_be16(ETH_P_ARP),
	.func =	arp_rcv,
};
```



+ **里面最重要的是`arp_rcv()`**



##### 2.4.1.3、static定义的arp_net_ops，pernet_operations

+ *这个是啥，没看到讲*
+ 是在net_namespace.h中，*暂时没看*

##### 2.4.1.4、notifier_block arp_netdev_notifier

+ `.notifier_call = arp_netdev_event,`

#### 2.4.2、初始化，arp_init

+ 在af_inet.c中调用
  + `inet_init()`中调用，注释写的是，`Set the ARP module up`，启动ARP模块
+ arp_init的实现过程
  + **函数的参数，都是2.4.1.2 -- 2.4.1.5的对象**
  + 初始化ARP协议的邻居表，`neigh_table_init()`
  + 协议栈中注册ARP协议，`dev_add_pack()`
  + `register_pernet_subsys()`不知道
  + 注册事件通知，`register_netdevice_notifier()`
    + dev.c里的注释，*todo，可以再细化一下*

#### 2.4.3、注册后，在arp_tbl中绑定的函数

##### 2.4.3.1、arp_constructor()，邻居项的初始化

+ *对应18.8*

+ **`arp_generic_ops()`也在这里面指定了**

##### 2.4.3.2、parp_redo，但封装的是arp_process

+ 

##### 2.4.3.3、arp_is_multicast

#### 2.4.4、neigh_ops结构中定义的函数

+ *对应18.7*

##### 2.4.4.1、arp_solicit，发送ARP请求

+ 调用`arp_send_dst()`，从而调用`arp_create()`

#### 2.4.5、arp_packet_type中的arp_rcv

##### 2.4.5.1、arp_rcv，从二层接收并处理一个报文，收到后还得处理，arp_process

+ Receive an arp request from the device layer.【注释】

+ `static int arp_rcv(struct sk_buff *skb, struct net_device *dev, struct packet_type *pt, struct net_device *orig_dev)`

  + skb，arp的报文
  + dev，接收arp报文的网络设备
  + pt，报文类型，*本处，参数，没用上吧*
  + orig_dev，接收arp报文的原始网络设备，*没用上*

+ 函数流程

  + `pskb_may_pull()`，检测报文完整性
    + ARP header, plus 2 device addresses, plus 2 IP addresses. 【注释，+2处设备地址，2个IP地址】，**核心在**，if_arp.h中的`arp_hdr_len()`完成了
  + `arp_hdr()`，arp报文的硬件地址与网络地址长度是否相同，协议地址长度是不是4
  + `memset(NEIGH_CB(skb)`，是把`skb->cb`给置0
  + `return NF_HOOK(NFPROTO_ARP, NF_ARP_IN, dev_net(dev), NULL, skb, dev, NULL, arp_process);`，转到arp_process处理
    + **这个NF_HOOK**，不太懂，todo

#### 2.4.6、收到后得处理，arp_process

+ *18.11.2*

##### 处理1、/* Update our ARP tables */

+ `arp_accept()`不太懂
+ `neigh_update()`

#### 2.4.7、arp_send_dst中的arp_create和arp_xmit

##### 2.4.7.1、arp_create

##### 2.4.7.2、arp_xmit

+ *就是18.10的，arp输出*
+ 里面`arp_xmit_finish()`干了啥呢，就是调用`dev_queue_xmit(skb)`，输出报文

```c
/*
 *	Send an arp packet.
 */
void arp_xmit(struct sk_buff *skb)
{
	/* Send it off, maybe filter it using firewalling first.  */
	NF_HOOK(NFPROTO_ARP, NF_ARP_OUT,
		dev_net(skb->dev), NULL, skb, NULL, skb->dev,
		arp_xmit_finish);
}
EXPORT_SYMBOL(arp_xmit);
```



#### 2.4.8、杂项

##### 2.4.8.1、代理

##### 2.4.8.2、ARP的ioctl，arp_ioctl，在arp_inet.c中调用

+ arp_inet.c中的`inet_ioctl()`调用了`arp_ioctl()`，**分各种case**

##### 2.4.8.3、外部事件，arp_netdev_event

+ 在`arp_init()`里就注册上了，`notifier_call`被指定了一下
+ *18.14中介绍了*

### 2.5、neighbour.c

+ *todo，它自身没啥主线逻辑，更多是靠外面调用*，**所以，目录是在net/core/neighbour.c**
+ 当然核心的观点是
  + 初始化，查找，清除neigh_tables，靠neigh_table_init和neigh_table_clear
    + `neigh_find_table()`，**区分IPV4和IPV6**

#### 2.5.1、邻居表的初始化，neigh_table_init

+ *todo，17.4*

+ **这个在`arp_init()`中第一行就被调用了**
+ `void neigh_table_init(int index, struct neigh_table *tbl)`，初始化tbl的各个字段
  + 这个index，不是理解的下标，**用于区别ARP还是ND**，也是ipv4和ipv6对应的
+ **最终填到neigh_tables结构体中**

#### 2.5.2、邻居项的初始化，neigh_init

+ *我个人的判断是通过`subsys_initcall()`告诉内核自己要初始化*
+ 功能是啥呢，通过`rtnl_register()`注册了5大函数
  + neigh_add
  + neigh_delete
  + neigh_get
  + neightbl_set

##### 2.5.2.1、neigh_add

+ *todo，17.6.1*
+ 逻辑的主线，如果add时没有，就需要创建`___neigh_create()`

##### 2.5.2.2、neigh_delete

##### 2.5.2.3、neigh_get

##### 2.5.2.4、neightbl_set

#### 2.5.3、邻居项的创建、查找、更新、散列表扩容

2.5.3.1、neigh_alloc

+ *17.7.1*

neigh_hash_grow

+ *17.8*

neigh_lookup

+ *17.9*

neigh_update

+ *17.10*
+ **arp_process**里用得比较多

#### 2.5.4、垃圾回收

2.5.4.1、neigh_forced_gc，同步回收

neigh_periodic_work

#### 2.5.5、定时器，neigh_timer_handler

+ 在`neigh_alloc()`调用

#### 2.5.6、代理项phash_buckets

##### 2.5.6.1、pneigh_lookup

+ *todo，17.1.4.1*

#### 2.5.7、输出与arp中的neigh_ops关联上

+ *这里的输出，为何分快慢呢*

##### 2.5.7.1、neigh_connected_output

+ *17.15.3*

#### 2.5.8、杂项

+ *17.5，邻居项的状态机，那个表不太熟啊*

## 最后

### 履历

+ 0629结合代码和书，把框架过了一遍，*细节要平时再看*

### next

+ 1、除了更好的理解数据结构（看注释的真实含义）、一些业务逻辑外（看rfc）
+ 2、arp_process需要整明白
+ 3、neigh_update需要整明白