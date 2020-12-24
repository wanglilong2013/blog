---
title: Golang并发控制
date: 2019-06-08 21:30:10
tags: [Go]
---

记录一下Golang并发控制的常用3种方法

## 1. 通过chinnel实现并发控制

无缓冲的通道指的是通道的大小为0，也就是说，这种类型的通道在接收前没有能力保存任何值，它要求发送 goroutine 和接收 goroutine
同时准备好，才可以完成发送和接收操作。

参考代码如下:

```
package main

import (
    "fmt"
)

func main() {
    //定义一个无缓冲通道
    ch := make(chan struct{})
    go func() {
        fmt.Println("做菜")
        ch<-struct{}{}
    }()
    go func() {
        fmt.Println("煮汤")
        ch<-struct{}{}
    }()
    <-ch
    <-ch
    fmt.Println("午饭做好了！")
}
```
## 2.通过sync.WatiGroup

主要利用WaitGroup在值递减至0之前调用Wait()方法会一致阻塞。 需要注意的一点是 wg 最好不要通过拷贝传递，不然容易出现死锁。

参考代码如下:

```
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    wg.Add(2)
    go func() {
        fmt.Println("做菜")
        wg.Done()
    }()
    go func() {
        fmt.Println("煮汤")
        wg.Done()
    }()
    wg.Wait()
    fmt.Println("午饭做好了！")
}
```
## 3. 通过Context

在Go 1.7 版本引进的强大的Context上下文，实现并发控制

参考代码如下:

```
package main

import (
    "fmt"
    "context"
    "time"
)

func main() {

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    go func() {
        select {
        case  <-ctx.Done():
            return
        default:
            fmt.Println("做菜")
        }
    }()
    go func() {
        // fmt.Println("煮汤")
        select {
        case <-ctx.Done():
            return
        default:
            fmt.Println("煮汤")
        }
    }()
    time.Sleep(50*time.Millisecond
    fmt.Println("午饭做好了！")
}

```


