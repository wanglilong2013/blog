---
title: Redis常用数据结构及使用场景
date: 2020-2-5 22:30:10
tags: [Redis]
---

Redis 常用数据结构

### string

字符串类型，经常用于k、v键值对缓存应用

### Hash
常用作对象存储，比如用来存储用户的一些简单信息

```
127.0.0.1:6379> HMSET user:100 name jack age 28 sex male
OK
127.0.0.1:6379> HGETALL user:100
1) "name"
2) "jack"
3) "age"
4) "28"
5) "sex"
6) "male"
```

### List(链表)
通常我们用lpush + rpop 的组合来时先队列功能， lpush + lpop 的组合来实现栈

```
127.0.0.1:6379> LPUSH orders 100 101 102
(integer) 3
127.0.0.1:6379> RPOP orders
"100"
127.0.0.1:6379> LPOP orders
"102"
127.0.0.1:6379> LPOP orders
"101"
```

### set(集合)

通过集合的交集功能来实现类似共同好友的功能, 例如id为1的好友有ID为2，5，7，8的用户，ID为2的用户有3，7，8用户，可以通过 sinter user:1:friends user:2:friends来计算它们的共同好友

```
127.0.0.1:6379> SADD user:1:friends 2 5 7 8
(integer) 4
127.0.0.1:6379> SADD user:2:friends 3 7 8
(integer) 4
127.0.0.1:6379> sinter user:1:friends user:2:friends
1) "7"
2) "8"
```

### sorted set(有序集合)

通过有序集合的score属性来实现排行榜, 这里我们用一个有序集合来存储文章的点击量，文章1 对应109点击次数，文章2对应102次点击，文章3对应12次点击，文章4对应87次点击，文章5对应9次点击，取点击次数最多的三篇文章对应为1，2，4.

详情如下:

```
127.0.0.1:6379> ZADD popular_posts  109 1 102 2 12 3 87 4 9 5
(integer) 5
127.0.0.1:6379> ZREVRANGE popular_posts 0 2
1) "1"
2) "2"
3) "4"

```