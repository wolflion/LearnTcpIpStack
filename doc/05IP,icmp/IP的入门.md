+ 要掌握一下**IP层的输入输出图**
  + 从网络设备层，`ip_rcv()`，dst_input再fowward或deliver
    + ip_forward()
    + ip_local_deliver()，到了**net_protocol->handler**
  + 从传输层过来，*是取skb队列里取嘛*，`ip_queue_xmit()`，到dst_output，再到ip_output，ip_finish_ouput，再到网络设备层



ip_init()是在ip_output.c中，分别调用了route.c中的`ip_rt_init()`和`inet_initpeers()`