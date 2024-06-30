主要还是fib_开头的文件

route.c【】-->fib_frontend.c【路由表的操作接口】-->fib_tire.c【路由表的维护】

ip_fib.h
fib_lookup.h，主要的结构是fib_alias

fib_hash.c，路由表的查找和维护  【废弃了】

fib_trie.c，
+ fib_table_insert()路由表的增加
+ fib_table_delte()路由表的删除

fib_frontend.c，实现操作路由表的接口函数和通知


那么route.c是干啥的呢？（路由缓存项的操作函数）
+ ip_rt_init()，这个是在 ip_output.c里的init()里调用它的
+ ip_rt_init()调用了哪些呢
	+ ip_fib_init()   【fib_frontend.c】
		+ 这里面有个fib_trie_init()，*也就是把hash换成trie了*
		+ *原来的kmalloc过程，封装成了一个函数fib_trie_table()了*



核心是**路由表的数据结构**


fib_frontend.c
+ ip_fib_init（）中，rtnl_register（）表示 route netlink，里面 inet_rtm_newroute，和删除路由  （为什么是netlink呢？）
+ inet_rtm_newroute()里会调用 fib_table_insert()
