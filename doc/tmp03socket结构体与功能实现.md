## 缘起

+ 看完第一轮后，发现**真正搞懂socket相关的结构**，有利于把各种点串起来
+ [Linux网络内核源码分析|套接字相关的数据结构和功能实现](https://xingkunz.github.io/2020/01/01/Linux%E7%BD%91%E7%BB%9C%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%A5%97%E6%8E%A5%E5%AD%97%E7%9B%B8%E5%85%B3%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E5%8A%9F%E8%83%BD%E5%AE%9E%E7%8E%B0/)，能看懂，看它这个也行，**我只是再整理一下**
+ 我基于原型是《Linux内核源码剖析-TCPIP》

  + chap10、Internet协议族
  + chap22-24、套接口层，套接口I/O，套接口选项，传输控制块
  
+ 自己看的代码是v6.6.9或更新

## 总结

+ *暂时不知道怎么写*

## 内容

###  chap10、Internet协议族，af_inet.c

+ socket编程时，知道**地址族AF**和**协议族PF**，*但实际中好像并不区分*
+ Internet协议族有具体哪些呢，**除了IPV4，IPV6外**
  + ICMP，TCP，UDP，ARP
+ 除了Internet协议族，还有哪些协议族
  + unix协议族，AF_UNIX，PF_UNIX，提供了IPC功能

#### net_proto_family结构，include/linux/net.h

+ 功能：从协议族到socket创建
  + 用`sock_register()`注册到**net_families**，【每个协议对应一个`inet_create()`或`packet_create()`】
    + 注册的实现是在net/socket.c中
  + `static const struct net_proto_family __rcu *net_families[NPROTO] __read_mostly;`    net/socket.c，**协议列表**

```c
struct net_proto_family {
	int		family;
	int		(*create)(struct net *net, struct socket *sock, int protocol, int kern);
	struct module	*owner;
};
```

+ 实例1，net/ipv4/af_inet.c
+ 实例2，net/ipv4/af_unix.c

#### inet_protosw结构，include/net/protocol.h

+ 功能：将prot与ops关联起来

  + 也是创建了一个数组**inetsw_array**，4种，IP，TCP，UDP，ICMP，同时**指定了具体的prot和ops**

  ```c
  /* Upon startup we insert all the elements in inetsw_array[] into
   * the linked list inetsw.
   */
  static struct inet_protosw inetsw_array[] =
  {
  	{
  		.type =       SOCK_STREAM,
  		.protocol =   IPPROTO_TCP,
  		.prot =       &tcp_prot,
  		.ops =        &inet_stream_ops,
  		.flags =      INET_PROTOSW_PERMANENT |
  			      INET_PROTOSW_ICSK,
  	},
  
  	{
  		.type =       SOCK_DGRAM,
  		.protocol =   IPPROTO_UDP,
  		.prot =       &udp_prot,
  		.ops =        &inet_dgram_ops,
  		.flags =      INET_PROTOSW_PERMANENT,
         },
  
         {
  		.type =       SOCK_DGRAM,
  		.protocol =   IPPROTO_ICMP,
  		.prot =       &ping_prot,
  		.ops =        &inet_sockraw_ops,
  		.flags =      INET_PROTOSW_REUSE,
         },
  
         {
  	       .type =       SOCK_RAW,
  	       .protocol =   IPPROTO_IP,	/* wild card */
  	       .prot =       &raw_prot,
  	       .ops =        &inet_sockraw_ops,
  	       .flags =      INET_PROTOSW_REUSE,
         }
  };
  ```

  

```c
/* This is used to register socket interfaces for IP protocols.  */
struct inet_protosw {
	struct list_head list;

    /* These two fields form the lookup key.  */
	unsigned short	 type;	   /* This is the 2nd argument to socket(2). */
	unsigned short	 protocol; /* This is the L4 protocol number.  */

	struct proto	 *prot;  //传输层协议
	const struct proto_ops *ops; //协议族套接字操作集
  
	unsigned char	 flags;      /* See INET_PROTOSW_* below.  */
};
```

#### net_protocol结构，include/net/protocol.h

+ 功能：网络层和传输层的桥梁，**根据传输层协议，找到处理方法**

  + 实例1，net/ipv4/af_inet.c中的**tcp_protocol**

  ```c
  static struct net_protocol tcp_protocol = {
      .handler = tcp_v4_rcv,
      .err_handler = tcp_v4_err,
      .no_policy = 1,
  }
  ```

  

```c
/* This is used to register protocols. */
struct net_protocol {
	int			(*handler)(struct sk_buff *skb);

	/* This returns an error if we weren't able to handle the error. */
	int			(*err_handler)(struct sk_buff *skb, u32 info);

	unsigned int		no_policy:1,
				/* does the protocol do more stringent
				 * icmp tag validation than simple
				 * socket lookup?
				 */
				icmp_strict_tag_validation:1;
	u32			secret;
};
```

#### 初始化，net/ipv4/af_inet.c

+ `fs_initcall(inet_init);`
  + todo1，*fs_initcall宏的展开不太熟*

### chap22、套接口层（先列出结构）

#### socket结构，include/linux/net.h

+ 功能：
  + **关联了struct sock和struct proto_ops**两个结构
  + 它的实例是唯一的

```c
/**
 *  struct socket - general BSD socket
 *  @state: socket state (%SS_CONNECTED, etc)
 *  @type: socket type (%SOCK_STREAM, etc)
 *  @flags: socket flags (%SOCK_NOSPACE, etc)
 *  @ops: protocol specific socket operations
 *  @file: File back pointer for gc
 *  @sk: internal networking protocol agnostic socket representation
 *  @wq: wait queue for several uses
 */
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

#### proto_ops结构，include/linux/net.h

+ 功能：**socket的操作函数**

```c
struct proto_ops {
	int		family;
	struct module	*owner;
	int		(*release)   (struct socket *sock);
	int		(*bind)	     (struct socket *sock,
				      struct sockaddr *myaddr,
				      int sockaddr_len);
	int		(*connect)   (struct socket *sock,
				      struct sockaddr *vaddr,
				      int sockaddr_len, int flags);
	int		(*socketpair)(struct socket *sock1,
				      struct socket *sock2);
	int		(*accept)    (struct socket *sock,
				      struct socket *newsock, int flags, bool kern);
	int		(*getname)   (struct socket *sock,
				      struct sockaddr *addr,
				      int peer);
	__poll_t	(*poll)	     (struct file *file, struct socket *sock,
				      struct poll_table_struct *wait);
	int		(*ioctl)     (struct socket *sock, unsigned int cmd,
				      unsigned long arg);
#ifdef CONFIG_COMPAT
	int	 	(*compat_ioctl) (struct socket *sock, unsigned int cmd,
				      unsigned long arg);
#endif
	int		(*gettstamp) (struct socket *sock, void __user *userstamp,
				      bool timeval, bool time32);
	int		(*listen)    (struct socket *sock, int len);
	int		(*shutdown)  (struct socket *sock, int flags);
	int		(*setsockopt)(struct socket *sock, int level,
				      int optname, sockptr_t optval,
				      unsigned int optlen);
	int		(*getsockopt)(struct socket *sock, int level,
				      int optname, char __user *optval, int __user *optlen);
	void		(*show_fdinfo)(struct seq_file *m, struct socket *sock);
	int		(*sendmsg)   (struct socket *sock, struct msghdr *m,
				      size_t total_len);
	/* Notes for implementing recvmsg:
	 * ===============================
	 * msg->msg_namelen should get updated by the recvmsg handlers
	 * iff msg_name != NULL. It is by default 0 to prevent
	 * returning uninitialized memory to user space.  The recvfrom
	 * handlers can assume that msg.msg_name is either NULL or has
	 * a minimum size of sizeof(struct sockaddr_storage).
	 */
	int		(*recvmsg)   (struct socket *sock, struct msghdr *m,
				      size_t total_len, int flags);
	int		(*mmap)	     (struct file *file, struct socket *sock,
				      struct vm_area_struct * vma);
	ssize_t 	(*splice_read)(struct socket *sock,  loff_t *ppos,
				       struct pipe_inode_info *pipe, size_t len, unsigned int flags);
	void		(*splice_eof)(struct socket *sock);
	int		(*set_peek_off)(struct sock *sk, int val);
	int		(*peek_len)(struct socket *sock);

	/* The following functions are called internally by kernel with
	 * sock lock already held.
	 */
	int		(*read_sock)(struct sock *sk, read_descriptor_t *desc,
				     sk_read_actor_t recv_actor);
	/* This is different from read_sock(), it reads an entire skb at a time. */
	int		(*read_skb)(struct sock *sk, skb_read_actor_t recv_actor);
	int		(*sendmsg_locked)(struct sock *sk, struct msghdr *msg,
					  size_t size);
	int		(*set_rcvlowat)(struct sock *sk, int val);
};
```



### chap24、传输控制块

#### proto结构，include/net/sock.h

+ 功能：socket层到传输层的接口，**在inet_protosw结构中**被关联

  + 实例1，net/ipv4/tcp_ipv4.c中的`struct proto tcp_prot{}`

  ```c
  struct proto tcp_prot = {
  	.name			= "TCP",
  	.owner			= THIS_MODULE,
  	.close			= tcp_close,
  	.pre_connect		= tcp_v4_pre_connect,
  	.connect		= tcp_v4_connect,
  	.disconnect		= tcp_disconnect,
  	.accept			= inet_csk_accept,
  	.ioctl			= tcp_ioctl,
  	.init			= tcp_v4_init_sock,
  	.destroy		= tcp_v4_destroy_sock,
  	.shutdown		= tcp_shutdown,
  	.setsockopt		= tcp_setsockopt,
  	.getsockopt		= tcp_getsockopt,
  	.bpf_bypass_getsockopt	= tcp_bpf_bypass_getsockopt,
  	.keepalive		= tcp_set_keepalive,
  	.recvmsg		= tcp_recvmsg,
  	.sendmsg		= tcp_sendmsg,
  	.splice_eof		= tcp_splice_eof,
  	.backlog_rcv		= tcp_v4_do_rcv,
  	.release_cb		= tcp_release_cb,
  	.hash			= inet_hash,
  	.unhash			= inet_unhash,
  	.get_port		= inet_csk_get_port,
  	.put_port		= inet_put_port,
  #ifdef CONFIG_BPF_SYSCALL
  	.psock_update_sk_prot	= tcp_bpf_update_proto,
  #endif
  	.enter_memory_pressure	= tcp_enter_memory_pressure,
  	.leave_memory_pressure	= tcp_leave_memory_pressure,
  	.stream_memory_free	= tcp_stream_memory_free,
  	.sockets_allocated	= &tcp_sockets_allocated,
  	.orphan_count		= &tcp_orphan_count,
  
  	.memory_allocated	= &tcp_memory_allocated,
  	.per_cpu_fw_alloc	= &tcp_memory_per_cpu_fw_alloc,
  
  	.memory_pressure	= &tcp_memory_pressure,
  	.sysctl_mem		= sysctl_tcp_mem,
  	.sysctl_wmem_offset	= offsetof(struct net, ipv4.sysctl_tcp_wmem),
  	.sysctl_rmem_offset	= offsetof(struct net, ipv4.sysctl_tcp_rmem),
  	.max_header		= MAX_TCP_HEADER,
  	.obj_size		= sizeof(struct tcp_sock),
  	.slab_flags		= SLAB_TYPESAFE_BY_RCU,
  	.twsk_prot		= &tcp_timewait_sock_ops,
  	.rsk_prot		= &tcp_request_sock_ops,
  	.h.hashinfo		= NULL,
  	.no_autobind		= true,
  	.diag_destroy		= tcp_abort,
  };
  EXPORT_SYMBOL(tcp_prot);
  ```

  

```c
/* Networking protocol blocks we attach to sockets.
 * socket layer -> transport layer interface
 */
struct proto {
	void			(*close)(struct sock *sk,
					long timeout);
	int			(*pre_connect)(struct sock *sk,
					struct sockaddr *uaddr,
					int addr_len);
	int			(*connect)(struct sock *sk,
					struct sockaddr *uaddr,
					int addr_len);
	int			(*disconnect)(struct sock *sk, int flags);

	struct sock *		(*accept)(struct sock *sk, int flags, int *err,
					  bool kern);

	int			(*ioctl)(struct sock *sk, int cmd,
					 int *karg);
	int			(*init)(struct sock *sk);
	void			(*destroy)(struct sock *sk);
	void			(*shutdown)(struct sock *sk, int how);
	int			(*setsockopt)(struct sock *sk, int level,
					int optname, sockptr_t optval,
					unsigned int optlen);
	int			(*getsockopt)(struct sock *sk, int level,
					int optname, char __user *optval,
					int __user *option);
	void			(*keepalive)(struct sock *sk, int valbool);
#ifdef CONFIG_COMPAT
	int			(*compat_ioctl)(struct sock *sk,
					unsigned int cmd, unsigned long arg);
#endif
	int			(*sendmsg)(struct sock *sk, struct msghdr *msg,
					   size_t len);
	int			(*recvmsg)(struct sock *sk, struct msghdr *msg,
					   size_t len, int flags, int *addr_len);
	void			(*splice_eof)(struct socket *sock);
	int			(*bind)(struct sock *sk,
					struct sockaddr *addr, int addr_len);
	int			(*bind_add)(struct sock *sk,
					struct sockaddr *addr, int addr_len);

	int			(*backlog_rcv) (struct sock *sk,
						struct sk_buff *skb);
	bool			(*bpf_bypass_getsockopt)(int level,
							 int optname);

	void		(*release_cb)(struct sock *sk);

	/* Keeping track of sk's, looking them up, and port selection methods. */
	int			(*hash)(struct sock *sk);
	void			(*unhash)(struct sock *sk);
	void			(*rehash)(struct sock *sk);
	int			(*get_port)(struct sock *sk, unsigned short snum);
	void			(*put_port)(struct sock *sk);
#ifdef CONFIG_BPF_SYSCALL
	int			(*psock_update_sk_prot)(struct sock *sk,
							struct sk_psock *psock,
							bool restore);
#endif

	/* Keeping track of sockets in use */
#ifdef CONFIG_PROC_FS
	unsigned int		inuse_idx;
#endif

#if IS_ENABLED(CONFIG_MPTCP)
	int			(*forward_alloc_get)(const struct sock *sk);
#endif

	bool			(*stream_memory_free)(const struct sock *sk, int wake);
	bool			(*sock_is_readable)(struct sock *sk);
	/* Memory pressure */
	void			(*enter_memory_pressure)(struct sock *sk);
	void			(*leave_memory_pressure)(struct sock *sk);
	atomic_long_t		*memory_allocated;	/* Current allocated memory. */
	int  __percpu		*per_cpu_fw_alloc;
	struct percpu_counter	*sockets_allocated;	/* Current number of sockets. */

	/*
	 * Pressure flag: try to collapse.
	 * Technical note: it is used by multiple contexts non atomically.
	 * Make sure to use READ_ONCE()/WRITE_ONCE() for all reads/writes.
	 * All the __sk_mem_schedule() is of this nature: accounting
	 * is strict, actions are advisory and have some latency.
	 */
	unsigned long		*memory_pressure;
	long			*sysctl_mem;

	int			*sysctl_wmem;
	int			*sysctl_rmem;
	u32			sysctl_wmem_offset;
	u32			sysctl_rmem_offset;

	int			max_header;
	bool			no_autobind;

	struct kmem_cache	*slab;
	unsigned int		obj_size;
	unsigned int		ipv6_pinfo_offset;
	slab_flags_t		slab_flags;
	unsigned int		useroffset;	/* Usercopy region offset */
	unsigned int		usersize;	/* Usercopy region size */

	unsigned int __percpu	*orphan_count;

	struct request_sock_ops	*rsk_prot;
	struct timewait_sock_ops *twsk_prot;

	union {
		struct inet_hashinfo	*hashinfo;
		struct udp_table	*udp_table;
		struct raw_hashinfo	*raw_hash;
		struct smc_hashinfo	*smc_hash;
	} h;

	struct module		*owner;

	char			name[32];

	struct list_head	node;
	int			(*diag_destroy)(struct sock *sk, int err);
} __randomize_layout;
```

#### proto注册、解注册，管理

#### sock_common、sock、inet_sock、tcp_sock、udp_sock

+ *todo，这部分要先用图来表示才更好*

### socket的流程

+ chap22、讲得不全（是从系统调用开始的，sys_connect）

#### 1、sock_init()，net/socket.c

+ *自己就记着，sock开头的，就先去socket.c里找*

#### 2、inet_init()，net/ipv4/af_inet.c

+ `fs_initcall(inet_init);`
  + [[linux内核initcall放置在各个section中函数执行流程](https://www.cnblogs.com/conscience-remain/p/17949742)

#### todo，后面再写

