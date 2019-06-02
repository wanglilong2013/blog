---
title: beanstalkd 协议篇
date: 2019-05-09 17:53:47
tags:
---

﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿# Beanstalkd 协议

## 说明
----

beanstalkd协议基于ASCII编码运行在tcp上。客户端连接服务器并发送指令和数据，然后等待响应并关闭连接。对于每个连接，服务器按照接收命令的序列依次处理并响应。所有整型值都非负的十进制数，除非有特别声明。

## Job的生命周期

一个工作任务job当client使用put命令时创建。在整个生命周期中job可能有四个工作状态：ready，reserved，delayed，buried。在put之后，一个job的典型状态是ready，在ready队列中，它将等待一个worker取出此job并设置为其为reserved状态。worker占有此job并执行，当job执行完毕，worker可以发送一个delete指令删除此job。


## job 状态说明

`ready`  等待去除并处理
`reserved` 如果job被worker取出，将被此worker预订，worker将执行此job
`delayed` 等待特定时间之后，状态再迁移为ready状态

Job 典型生命周期

```
put            reserve               delete
  -----> [READY] ---------> [RESERVED] --------> *poof*
```

Job 可能的状态迁移


```
   put with delay               release with delay
  ----------------> [DELAYED] <------------.
                        |                   |
                 kick   | (time passes)     |
                        |                   |
   put                  v     reserve       |       delete
  -----------------> [READY] ---------> [RESERVED] --------> *poof*
                       ^  ^                |  |
                       |   \  release      |  |
                       |    `-------------'   |
                       |                      |
                       | kick                 |
                       |                      |
                       |       bury           |
                    [BURIED] <---------------'
                       |
                       |  delete
                        `--------> *poof*
```

## Tubes

一个服务器有一个或者多个tubes，用来储存统一类型的job。每个tube由一个就绪队列与延迟队列组成。每个job所有的状态迁移在一个tube中完成。consumers消费者可以监控感兴趣的tube，通过发送watch指令。consumers消费者可以取消监控tube，通过发送ignore命令。通过watch list命令返回所有监控的tubes，当客户端预订一个job，此job可能来自任何一个它监控的tube。

当一个客户端连接上服务器时，客户端监控的tube默认为defaut，如果客户端提交job时，没有使用use命令，那么这些job就存于名为default的tube中。

tube按需求创建，无论他们在什么时候被引用到。如果一个tube变为空（即no ready jobs，no delayed jobs，no buried jobs）和没有任何客户端引用，它将会被自动删除。


