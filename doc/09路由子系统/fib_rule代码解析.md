fib_rules.c分别在2个目录下都有，

+ ipv4/fib_rules.c，处理IPv4网络层的FIB规则。
+ net/core/fib_rules.c，处理通用的、与网络层协议无关的FIB规则。

### fib_rules.c【ipv4】

#### 结构体fib4_rule

+ **在fib_rule的基础上，增加了几个字段**

#### 初始化fib4_rules_init()

##### fib4_rules_init

+ fib_frontend.c中的`ip_fib_net_init()`，**但它自身也有个static的fib4_rules_init()**
+ 函数过程
  + 1、调用core中的`fib_rules_register()`，**参数是，fib_rules_ops的对象fib4_rules_ops_template**
  + 2、调用`fib_default_rules_init()`
  + *net->ipv4.rules_ops*这个对应哪个结构体？

##### fib_default_rules_init

+ 分了local，main，default
+ 函数都是core/*.c中的`fib_default_rule_add()`

#### ops对应的函数

fib4_rule_action

fib4_rule_suppress

fib4_rule_match

fib4_rule_configure

fib4_rule_delete

fib4_rule_compare

fib4_rule_fill

fib4_rule_nlmsg_payload

fib4_rule_flush_cache

#### 没用到的函数，也可能是外面的接口

fib4_rule_default

+ *没找到*

fib4_rules_dump

+ fib_notifier.c中调用

__fib_lookup

+ *没找到*

fib4_rules_seq_read

+ fib_notifier.c中调用

#### 被本文件中调用的

fib4_rule_matchall

+ fib4_rule_default()调用

fib_empty_table

+ fib4_rule_configure()调用

### ip_rules.c【core】

#### 结构体

##### fib_rules_net_ops

#### 初始化

##### fib_rules_init

+ **通过subsys_initcall()，自己向内核注册**

#### 初始化后的函数

fib_nl_newrule，新规则

fib_nl_delrule，删规则

fib_nl_dumprule，获取规则

fib_rules_net_init

fib_rules_net_exit

fib_rules_event

#### 被本文件中调用的

detach_rules

+ fib_rules_event

attach_rules

+ fib_rules_event

rule_exists

+ fib_nl_newrule

fib_default_rule_pref

+ fib_nl2rule

fib_nl2rule

+ fib_nl_newrule

lookup_rules_ops

+ fib_rules_dump

#### 被外面调用

##### fib_rule_matchall

+ ipv4里面的fib4_rule_matchall

##### fib_default_rule_add

+ ipv4里面的fib_default_rules_init

### fib_notifier.c【ipv4】

#### 结构体fib4_notifier_ops_template

#### 初始化

##### fib4_notifier_init

+ fib_frontend.c中
+ 过程
  + 调用core中的`fib_notifier_ops_register()`

#### 初始化后函数

##### fib4_seq_read

##### fib4_dump

### 在外面调用的

##### call_fib4_notifier

+ 在fib_trie.c中【call_fib_entry_notifier】

##### call_fib4_notifiers

+ 在fib_trie.c中【call_fib_entry_notifiers】