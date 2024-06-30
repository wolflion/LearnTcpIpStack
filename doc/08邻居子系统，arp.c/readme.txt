net/ipv4/arp.c

neigh_ops， C:\2Code\linux-6.6.9\include\net\neighbour.h  【但凡是ops，是不是大多数都是 回调函数】

3个变量，arp_generic_ops和arp_hh_ops和arp_direct_ops，区别以及应用场景在哪？

neigh_table， C:\2Code\linux-6.6.9\include\net\neighbour.h


struct neighbour，这个在arp_solicit()中用到；








https://www.cnblogs.com/lizhuming/p/16845687.html  【参考：【lwip】08-ARP协议一图笔记及源码实现 】

