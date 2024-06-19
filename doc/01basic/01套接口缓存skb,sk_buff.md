## 缘起

## 二、内容

### 2.1、为何引用Socket Buffer？它是啥？

+ Socket Buffer 就是要接收或发送的网络数据包在内核中的对象实例。

### 2.2、Socket Buffer 的构成

+ skbuff.h 
+ 一个完整的 Socket Buffer 主要由两个实体组成
  + 数据包：存放实际要在网络中传送的数据缓冲区
  + 管理数据结构（struct sk_buff）：当在内核中处理数据包时，内核还需要一些其他的数据来管理数据包和操作数据包，例如数据接收/发送的时间、状态等。
+ 网络数据包在 TCP/IP 协议栈的传送示意图，**图2-3**

### 2.3、详解sk_buff结构

+ sk_buff 数据域分类
  + 结构管理
  + 常规数据域
  + 网络功能配置相关域

#### 2.3.3、sk_buff 的网络功能配置域

### 2.4、操作 sk_buff 的函数

+ 从其功能上可以划分为以下几类
  + 创建、释放和复制 Socket Buffer
  + 操作 sk_buff 结构中的参数和指针
  + 管理 Socket Buffer 的队列

#### 2.4.1 创建和释放 Socket Buffer

+ `__alloc_skb`

#### 2.4.2 数据空间的预留和对齐

+ skb_reserve

#### 2.4.3 复制和克隆

#### 2.4.4 操作队列的函数

#### 2.4.5 引用计数的操作

+ skb_get()

#### 2.4.6 协议头指针操作

### 2.5、skb_shared_info支持IP分片和TCP分段

+ struct skb_shared_info 是用于支持 IP 数据分片（fragmentation）和 TCP 数据分段（segmentation）的数据结构。

#### 2.5.2 设计 skb_shared_info 数据结构的目的

#### 2.5.3 操作 skb_shared_info 的函数

## 三、最后

### 总结

### ref

+ 1、Linux内核源码剖析-tcpip实现(上)，chap3套接口缓存（sk_buff）
+ 2、嵌入式Linux网络体系结构设计与TCPIP协议栈，chap2 Socket Buffer