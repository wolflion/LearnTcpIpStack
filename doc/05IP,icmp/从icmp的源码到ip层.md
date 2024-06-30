

## 内容

### 2.1、准备工作

#### 2.1.1、ICMP报文

+ 对应include/uapi/linux/icmp.h中的`struct icmphdr{}`，**为何是uapi呢，表示User-space**

#### 2.1.2、注册ICMP报文类型

+ af_inet.c中的`icmp_protocol`，是个net_protocol类型，*我记成注册，packet_type类型了*，里面绑定`icmp_rcv`和`icmp_error`

### 2.2、icmp.c

#### 2.2.1、初始化static的数组

##### 2.2.1.1、pernet_operations

##### 2.2.1.2、struct icmp_control icmp_pointers ，ICMP控制数组

+ 指定了哪些类型，对应了哪些处理函数，icmp_discard，ping_rcv

#### 2.2.2、初始化icmp_init

+ *也有pernet_operations*，这个是在 netnamespce.h中
+ **最终调用`icmp_sk_init()`**，初始化ipv4的系统参数？
  + net->ipv4.sysctl_icmp_echo_ignore_all，用于确定接收或忽略请求回显报文
  + *todo，书中，14.3有完整介绍*

#### 2.2.3、初始化后，要处理进来的ICMP包，imcp_rcv

+ *广播和组播啥意思，只是设置一个标置吧，具体还是由icmp_pointers绑定的处理？*
+ 当然里面调用了icmp_echo，ping_rcv

#### 2.2.4、既然进来和处理了，要发送出去的吧，__icmp_send

+ *问题是，没看到哪调用啊*
+ https://elixir.bootlin.com/linux/latest/source/net/ipv4/route.c#L1233  `ipv4_send_dest_unreach()`这里面有调用

#### 2.2.5、icmp_pointers里关联的函数

#### 2.2.6、其它

### 2.3、ping.c

#### 2.3.1、从ping_rcv而来

+ *看样子这个函数是独立的*

#### 2.3.2、协议类型的struct proto ping_prot及对应的函数

+ 跟udp，tcp一样，在af_inet.c中的inet_protosw inetsw_array里定义好了
+ static开头的函数，应该是只在本部分调用
+ *从第一个init开始吧*

##### 2.3.2.1、ping_init_sock

##### ping_v4_sendmsg

+ 这里面有个`ipcm_init_sk()`关联到了ip.h上