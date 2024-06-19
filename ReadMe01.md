+ 1、基础知识  【*之前起了个头，2024-03-21开始写点内容*】
  + sk_buff的结构，以及引入
  + net_device的结构，以及引入
+ 2、专题
  + 2.1、先从**套接字层**开始，有用户态，到内核态  【3.20起了个头，*后面要不断复习*】
  + 2.2、想从**路由子系统**下手，*感觉相对比较独立？*  【3.24-3.30那周】



## 相关人员的blog

+ xingkunz.github.io，linux网络内核源码分析
+ switch-router.gitee.io
  + 写过一篇，*如何学习linux内核网络协议栈*，我竟然能看懂了
  + struct socket，是向上，面向用户
  + struct sock，是向下，面向协议栈

## 环境搭建上的

+ https://blog.csdn.net/weixin_49398773/article/details/135216001 ，TCP/IP协议栈源代码分析：GDB调试环境搭建及源码分析
+ https://zhuanlan.zhihu.com/p/587102010  ，Ubuntu22.04上实现GDB+Qemu调试Linux内核网络协议栈的环境配置教程
+ https://blog.csdn.net/weixin_54132956/article/details/133965188  在Ubuntu22.04上搭建Linux0.11内核实验环境详细教程，*只是make成一个包吧，不能开发和调试吧*