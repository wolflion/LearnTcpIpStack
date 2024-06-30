## 缘起

+ 11-13章，网际协议，IP选项处理，IP的分片与组装，
+ 15章，IP组播
+ 《深入内幕》里面  Part5讲的就是IPv4，chap18开始，18.1的图不错，**准备自己画一个，也顺带理解一下**

## 内容

### chap11、网际协议

+ 是不是都是ip_*.c表示相关的
+ **图11-3**，IP层主要函数调用关系，**是从`dst->ouput()`或者`dst->input()`**，作者这个图比18.1那个图好理解一些
+ 问题1
  + include/net/ip.h与include/linux/ip.h两者有何区别不？

#### 问题1、IP层有没有啥特别的数据结构？

#### 11.4、初始化，ip_init()，net/ipv4/ip_output.c

+ *这个output和input，怎么去区分？分别对应谁，才是input和ouput？*
  + input是从**设备层来**，往**网络层UDP，TCP去**

+ **由net/ipv4/af_inet.c中的`inet_init()`调用**，*问题是，这2层是啥关系【这个应该是层，传输层的上层呢】*
+ *问题2，传输层怎么跟IP层交互呢，是通过skb吗？*

#### 11.10、输入，ip_rcv()，net/ipv4/ip_input.c

+ *问题todo*，**是不是可以理解为在af_inet.c中已经做好一切准备工作了**，如果是确定用的是internet协议族的话
+ af_inet.c中定义了**ip_packet_type**
  + 这个是`inet_init()`里给add进来了，`dev_add_pack(&ip_packet_type);`
    + dev_add_pack()函数是，net/core/dev.c中的，*没看懂*

```c
static struct packet_type ip_packet_type __read_mostly = {
	.type = cpu_to_be16(ETH_P_IP),
	.func = ip_rcv,
	.list_func = ip_list_rcv,
};
```

+ 最终会分2个方向，**转发，还是输入到本机**
  + 输入到本机，`ip_local_deliver()`，Deliver IP Packets to the higher protocol layers.**是往TCP/UDP发**
  + 转发，`ip_forward()`，**有个ip_forward.c文件**
    + 问题，谁来调用`ip_forward()`，逻辑应该是ip_input.c里的，但我没找到，**书上写的是，ip_rcv_finish()中调用的**
    + 调用1，net/ipv4/route.c中的`rth->dst.input = ip_forward;`
    + 调用2，net/ipv4/devinet.c中的`static struct ctl_table ctl_forward_entry[]`

#### 11.11、输出，ip_output()，是指输出到网络设备

+ *问题，为啥要有这种区分？*，三种都在net/ipv4/output.c中定义了

##### 11.11.1、ip数据报输出到设备，ip_output()

+ ip_output.c中的还是有`ip_output()`函数的，最终走向include/net/dst.h中的定义的`static inline int dst_output(struct net *net, struct sock *sk, struct sk_buff *skb)`，用的是**`skb_dst(skb)->output,`**

##### 11.11.2、TCP输出的接口，ip_queue_xmit()，net/ipv4/output.c

##### 11.11.3、UDP输出的接口，ip_append_data()，net/ipv4/output.c

### chap12、IP选项处理，ip_options.c

+ 暂时先不看，主要有rfc里有介绍，也有单独的实现文件，*问题是，如何把这些函数串起来，自己暂时没思考过*

### chap13、IP的分片与组装，ip_fragment.c

+ 但IP分片的业务逻辑还是在`ip_fragment()`（net/ipv4/output.c）中
+ rfc791的14/51里提到了Fragmentation

#### 13.2、分片

+ *问题1，分片的逻辑* ，当然分为**快速**与**慢速**

#### 13.3、组装

+ *问题2，组装的逻辑*

##### 1、用什么方式去管理（ipq_hash）

+ *ipq的方式，是不是废弃啦*

+ *我估计用，include/net/inet_frag.h中的管理了*