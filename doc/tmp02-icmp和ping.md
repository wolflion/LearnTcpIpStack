af_inet.c中的ICMP，对应了 ping_prot，这个地方是在ping.c中实现的net/ipv4/ping.c



但问题是icmp.c里，难道就跟这个没关系了吗？

> **icmp.c与ping_prot的关系**
>
> - icmp.c是Linux内核中的一个文件，它实现了ICMP协议。
> - ping_prot是Linux内核中的一个文件，它实现了ping命令。
>
> ping命令使用ICMP协议来检测网络连接的状况。当用户执行ping命令时，ping_prot会向目标主机发送ICMP回显请求报文。如果目标主机收到ICMP回显请求报文，它会发送ICMP回显应答报文。ping_prot会根据ICMP回显应答报文来判断网络连接的状况。
>
> icmp.c和ping_prot都是Linux内核中的重要文件，它们共同作用来实现ping命令。



af_inet.c中有个`icmp_init()`



`icmp_pointers` This table is the definition of how we handle ICMP.  **定义了一个映射关系**

> #include <linux/icmp.h>
> 定义一个指向 icmp_pointers 数组的指针，命名为 icmp_func：
> c
> 复制
> void (*icmp_func[])(struct sk_buff *) = icmp_pointers;
> 获取 ICMP 报文的类型（icmp_type）：
> c
> 复制
> unsigned int icmp_type = icmp_hdr(skb)->type;
> 使用获取到的 ICMP 类型作为索引，通过 icmp_func 调用相应的处理函数：
> c
> 复制
> icmp_func[icmp_type](skb);





### 主线知道了

+ af_inet.c中的`icmp_rcv()`，	*问题是，inet_init()中哪个 先呢，与icmp_init()时*
  + 这里面有个`reason = icmp_pointers[icmph->type].handler(skb);`

+ 定义了`__icmp_send()`，被**icmp_ndo_send**
  + 用宏`#if IS_ENABLED(CONFIG_NF_NAT)`
  + 提供给别人使用了，比如net/ipv4/route.c中验证 dst是否可达
+ icmp_echo()中调用了`icmp_replay()`
  + icmp_replay()再调用`icmp_push_replay()`