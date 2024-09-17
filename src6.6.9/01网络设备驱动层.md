## 缘分

## 内容

### 设备驱动层

#### dev.c是设备层通用

##### 初始化

+ 1、入口就是`subsys_initcall(net_dev_init);`
+ 2、函数`net_dev_init()`
  + 樊东东（FDD）的4.4节，**网络设备处理层初始化**
  + 2.1、
  + 2.2、`netdev_kobject_init()`用到了`net/core/net-sysfs.c`的目录，*这是啥相关基础知识*
  + 2.3、*怎么让不空呢，开始时没人往里面add啊*，`ptype_base`数组
  + 2.4、`register_pernet_subsys`这个是`net_namespace.c`中定义的，也是`net/core`目录的
+ 3、函数`netdev_init()`
  + 被2.4给调用了，是`netdev_net_ops`定义的
  + *这个net_namespace是新的，老的源码阅读里，就没有它*
+ 总结，*1-3，表示初始化就结束了*

##### 二、注册（真实的驱动调用register_netdev，e100_probe）

+ 2.1、函数`register_netdev()`
  + FDD书的5.3.3节，**网络设备注册过程**
  + 这是一个包裹函数
+ 2.2、函数`register_netdevice()`，*内容还是蛮多的*

#### e100.c是具体的网卡

##### 初始化（）

##### 注册（会调用dev.c中的register_netdev）