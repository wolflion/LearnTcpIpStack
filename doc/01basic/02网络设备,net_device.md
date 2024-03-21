## 缘起

## 内容

### 2.1、协议栈与网络设备

#### 2.1.1 协议栈软件与网络设备硬件之间的接口

+ **图3-2**，比较重要
  + 上层协议栈
  + 独立于设备的接口文件dev.c
    + netif_rx（），操作skb
  + 硬件设备抽象 net_device
    + dev->hard_start_xmit
    + dev->open
    + dev->stop
  + 与硬件设备细节相关部分，**网络设备驱动程序**
    + net_rx，也操作skb
    + net_dev_stop
    + netdev_open

#### 2.1.2 设备独立接口文件 dev.c

#### 3.1.4 struct net_device 数据结构

### 2.2、struct net_device 数据结构

+ *数据域的划分*，以及*函数指针*