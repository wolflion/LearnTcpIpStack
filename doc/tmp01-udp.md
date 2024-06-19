## 解决了啥问题

+ 1、《LINUX内核源码剖析》下册chap33中各个小节是怎么串起来的
+ 2、核心文件是net/ipv4/tcp.c

## 内容

### 源起（inetsw_array[]），net/ipv4/af_inet.c

+ 核心部分（目前有4部分，还有IP，TCP、ICMP）

```c
{
    .type =       SOCK_DGRAM,
	.protocol =   IPPROTO_UDP,
	.prot =       &udp_prot,
	.ops =        &inet_dgram_ops,
	.flags = INET_PROTOSW_PERMANENT,
},
```

+ net/ipv4/af_inet.c中的`static struct inet_protosw inetsw_array[]`

  + 注释，Upon startup we insert all the elements in inetsw_array[] into the linked list inetsw.  启动后会把元素插入到**inetsw链表**

    + 【怎么实现的呢？】`inet_init()`中，通过

    ```c
    for (q = inetsw_array; q < &inetsw_array[INETSW_ARRAY_LEN]; ++q)
    		inet_register_protosw(q); // 注册进来  【todo，这个细节后面看下】
    ```

+ 【问题：udp_prot和inet_dgram_ops对应的结构体是`struct proto`和`struct proto_ops`，两者有啥区别与联系？】

  + *我之前的理解是，udp_prot对着的是数据，inet_dgram_ops对应的是属性，尤其是虚函数*

  + **这个比较好的理解实现是`sock->ops->sendmsg()`，这个其实是`inet_sendmsg()`**

    + `sock->ops`对应的是`proto_ops`中的`send_msg()`，**也就是`inet_sendmsg`**
    + **net/socket.c中的`sock_sendmsg()`**，是不是可以理解为*sock_开头的函数，都在socket.c中*
    + **代码中`sk->sk_prot->sendmsg`，后面关联是udp_sendmsg** 【*书上这么写的，自己找源头找到了*】
      + https://elixir.bootlin.com/linux/v6.9.5/source/include/net/sock.h#L372，`#define sk_prot			__sk_common.skc_prot`  ，也就是`sock_common`里的`struct proto`，**这个对方就是对应的proto结构体**

    ```c
    int inet_sendmsg(struct socket *sock, struct msghdr *msg, size_t size)
    {
    	struct sock *sk = sock->sk;
    	if (unlikely(inet_send_prepare(sk)))
    		return -EAGAIN;
        //INDIRECT_CALL_2 是一个宏定义，用于在 Linux 内核中执行间接函数调用。
        //include/linx/indirect_call_wrapper.h，表示要么执行udp_sendmsg，要么是tcp_send，取决于第1个参数的值是啥？
    	return INDIRECT_CALL_2(sk->sk_prot->sendmsg, tcp_sendmsg, udp_sendmsg,
    			       sk, msg, size);  // todo？为什么2个都写了呢？
    }
    EXPORT_SYMBOL(inet_sendmsg);
    ```

    

  + `struct proto` 是定义在 `include/net/sock.h` 头文件中的结构体

    + Networking protocol blocks we attach to sockets. socket layer -> transport layer interface  【socket层到传输层的接口】，*网络协议与socket的绑定？*
    + 【疑问：它会关联上proto_ops吗？】

  + `struct proto_ops` 是定义在 `include/linux/net.h` 头文件中的结构体

    + **这是跟socket结构体绑定的**

    ```c
    struct socket {
    	socket_state		state;
    	short			type;
    	unsigned long		flags;
    	struct file		*file;
    	struct sock		*sk;
    	const struct proto_ops	*ops; /* Might change with IPV6_ADDRFORM or MPTCP. */
    	struct socket_wq	wq;
    };
    ```

    

  + 问题了下AI

    + `struct proto_ops` 定义了网络协议操作的接口函数集合，而 `struct proto` 则用于描述和管理网络协议本身【协议的基属性和状态】。
    + *自己的疑惑是，两者的操作差不多啊*

    > 关系：
    > udp_prot 使用了 inet_dgram_ops 结构体中定义的操作函数集合来实现 UDP 协议的具体行为。通过将 inet_dgram_ops 结构体中的函数指针赋值给 udp_prot 结构体中相应的字段，实现了 UDP 协议的操作接口。这种设计使得 UDP 协议能够在 IPv4 网络协议栈中进行正确的初始化和处理。
    >
    > 因此，可以说 inet_dgram_ops 提供了面向数据报的套接字操作函数集合，而 udp_prot 则使用了这些函数来实现 UDP 协议的具体行为。它们共同协作，使得 UDP 协议能够在 IPv4 网络中正常运行。

### udp_prot实例，net/ipv4/udp.c

+ 核心部分（书中就讲了后面的代码）

```c
struct proto udp_prot = {
	.name			= "UDP",
	.owner			= THIS_MODULE,
	.close			= udp_lib_close,
	.pre_connect	= udp_pre_connect,
	.connect		= ip4_datagram_connect,
	.disconnect		= udp_disconnect,
	.ioctl			= udp_ioctl,
	.init			= udp_init_sock,
	.destroy		= udp_destroy_sock,
	.setsockopt		= udp_setsockopt,
	.getsockopt		= udp_getsockopt,
	.sendmsg		= udp_sendmsg,
	.recvmsg		= udp_recvmsg,
	.splice_eof		= udp_splice_eof,
	.release_cb		= ip4_datagram_release_cb,
	.hash			= udp_lib_hash,
	.unhash			= udp_lib_unhash,
	.rehash			= udp_v4_rehash,
	.get_port		= udp_v4_get_port,
	.put_port		= udp_lib_unhash,
#ifdef CONFIG_BPF_SYSCALL
	.psock_update_sk_prot	= udp_bpf_update_proto,
#endif
	.memory_allocated	= &udp_memory_allocated,
	.per_cpu_fw_alloc	= &udp_memory_per_cpu_fw_alloc,

	.sysctl_mem		= sysctl_udp_mem,
	.sysctl_wmem_offset	= offsetof(struct net, ipv4.sysctl_udp_wmem_min),
	.sysctl_rmem_offset	= offsetof(struct net, ipv4.sysctl_udp_rmem_min),
	.obj_size		= sizeof(struct udp_sock),
	.h.udp_table		= NULL,
	.diag_destroy		= udp_abort,
};
EXPORT_SYMBOL(udp_prot);
```

### inet_dgram_ops实例，net/ipv4/af_inet.c

+ 核心部分

```c
const struct proto_ops inet_dgram_ops = {
	.family		   = PF_INET,
	.owner		   = THIS_MODULE,
	.release	   = inet_release,
	.bind		   = inet_bind,
	.connect	   = inet_dgram_connect,
	.socketpair	   = sock_no_socketpair,
	.accept		   = sock_no_accept,
	.getname	   = inet_getname,
	.poll		   = udp_poll,
	.ioctl		   = inet_ioctl,
	.gettstamp	   = sock_gettstamp,
	.listen		   = sock_no_listen,
	.shutdown	   = inet_shutdown,
	.setsockopt	   = sock_common_setsockopt,
	.getsockopt	   = sock_common_getsockopt,
	.sendmsg	   = inet_sendmsg,
	.read_skb	   = udp_read_skb,
	.recvmsg	   = inet_recvmsg,
	.mmap		   = sock_no_mmap,
	.splice_eof	   = inet_splice_eof,
	.set_peek_off	   = sk_set_peek_off,
#ifdef CONFIG_COMPAT
	.compat_ioctl	   = inet_compat_ioctl,
#endif
};
EXPORT_SYMBOL(inet_dgram_ops);
```

### udp_sock实例，以及管理

#### udp_sock管理，include/net/udp.h中的`struct udp_table`

### udp_rcv和udp_err ，net/ipv4/af_inet.c

+ 核心，**还是在`inet_init()`中**

```c
	net_hotdata.udp_protocol = (struct net_protocol) {
		.handler =	udp_rcv,
		.err_handler =	udp_err,
		.no_policy =	1,
	};
```

#### udp_rcv

+ 在net/ipv4/ip_input.c中进行了声明`INDIRECT_CALLABLE_DECLARE(int udp_rcv(struct sk_buff *));`



### 问题

+ udp_poll()怎么关联上呢？ **总要有个`sock->ops`的实例指向epoll吧**

> 当应用程序在使用 epoll 监听 UDP 套接字时，内核会根据套接字的类型和协议族，选择对应的套接字操作函数集合。对于 UDP 套接字，在 IPv4 网络中使用的是 `inet_dgram_ops`。在 `inet_dgram_ops` 中，有一个 `poll` 函数指针，指向处理套接字轮询事件的函数。
>
> 当需要对 UDP 套接字进行轮询时，内核会调用 `udp_poll()` 函数，然后通过 `sock->ops->poll` 调用 `inet_dgram_ops` 中的 `poll` 函数，实现对应的轮询事件处理。
>
> 综上所述，通过 `sock->ops` 中的函数指针，特别是 `poll` 函数指针，UDP 套接字能够与 epoll 相关联，实现对套接字的轮询事件处理。



af_inet.c中的`init()`里也调用了udp_init()