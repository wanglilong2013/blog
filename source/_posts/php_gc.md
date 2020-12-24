---
title: PHP垃圾回收
date: 2018-03-12 22:30:10
tags: [PHP]
---

今天主要探讨一下PHP的垃圾回收机制，PHP作为世界上最好的语言，它在内存管理这一块是怎样做的呢？ 其实现在大多数语言在内存管理这一块都不需要我们去处理，为什么呢？
很重要的一点是因为内存的分配与回收，稍不注意就会产生内存泄漏。PHP的垃圾回收主要主要使用了引用计数


## 1. 引用计数基本知识
每个PHP变量存在一个叫“zval” (zend value)的变量容器中。这个变量容器除了包含变量的类型和值以外，还包括两个字节的额外信息，第一个是`is_ref`，是一个bool值，用来标识
这个变量是否属于引用集合，用以和普通变量区分开来。另外一个字节是`refcount`，用来标识指向这个变量容器的变量个数。

### 生成一个新的zval变量容器

```
<?php
$a = "new string";
xdebug_debug_zval('a');
```
以上程序会输出：
```
a: （refcount=1, is_ref=0）= 'new string'
```

### 增加一个zval的引用计数

```
<?php
$a = "new string";
$b = $a;
xdebug_debug_zval('a');
```

程序输出：
```
a: (refcount=2, is_ref=0) = 'new string'
```

### 减少zval的引用计数

```
<?php
$a = "new string";
$c = $b = $a;
xdebug_debug_zval('a');
unset($b, $c);
xdebug_debug_zval('a');
```

以上程序输出：

```
a: (refcount=3, is_ref=0) = 'new string'
a: (refcount=1, is_ref=0) = 'new string'
```
此时如果我们再执行`unset($a)；`,包含类型和值的这个变量容器就会从内存中删除。


## 1. 复合类型

### 创建一个array zval

```
<?php
$a = ['meaning' => 'life', 'number' => 42];
xdebug_debug_zval('a');
```

以上程序输出:

```
a: (refcount=1, is_ref=0) = array(
    'meaning' => (refcount=1, is_ref=0) = 'life',
    'number' => (refcount=1, is_ref=0) = 42
)
```

如图：
![pic](/imgs/arr1.png)

### 添加一个已经存在的元素到数组中

```
<?php
$a = ['meaning' => 'life', 'number' => 42];
$a['life'] = $a['meaning'];
xdebug_debug_zval('a'); 

```

以上程序输出：

```
a: (refcount=1, is_ref=0) => array(
    'meaning' => (refcount=2, is_ref=0) = 'life',
    'number' => (refcount=1, is_ref=0) = 42,
    'life' => (refcount=2, is_ref=0) => 'life'
)
```

图示:
![pic](/imgs/arr2.png)

### 从数组中删除一个元素

```
<?php
$a = ['meaning' => 'life', 'number' => 42];
$a['life'] = $a['meaning'];
unset($a['meaning'], $a['number']);
xdebug_debug_zval('a');
```

以上程序输出:

```
a: (refcount=1, is_ref=0)=> array(
    'life' => (refcount=1, is_ref=0) = 'life'
)
```

### 把数组作为一个元素添加到自己

```
<?php
$a = ['one'];
$a[] = &$a;
xdeug_debug_zval('a');
```

以上程序输出:

```
a: (refcount=2, is_ref=1) => array(
    0 => (refcount=1, is_ref=0) = 'one',
    1 => (refcount=2, is_ref=1) = ...
)
```

图示:
![pic](/imgs/arr3.png)

能看到数组变量 (a) 同时也是这个数组的第二个元素(1) 指向的变量容器中“refcount”为 2。上面的输出结果中的"..."说明发生了递归操作, 显然在这种情况下意味着"..."指向原始数组。

跟刚刚一样，对一个变量调用unset，将删除这个符号，且它指向的变量容器中的引用次数也减1。所以，如果我们在执行完上面的代码后，对变量$a调用unset, 那么变量 $a 和数组元素 "1" 所指向的变量容器的引用次数减1, 从"2"变成"1". 下例可以说明:

示例 Unsetting $a

```
(refcount=1, is_ref=1) = array (
    0 => (refcount=1, is_ref=0) => 'one',
    1 => (refcount=1, is_ref=1) => ...
)
```

图示:
![pic](/imgs/arr4.png)

尽管不再有某个作用域中的任何符号指向这个结构(就是变量容器)，由于数组元素“1”仍然指向数组本身，所以这个容器不能被清除 。因为没有另外的符号指向它，用户没有办法清除这个结构，结果就会导致内存泄漏。庆幸的是，php将在脚本执行结束时清除这个数据结构，但是在php清除之前，将耗费不少内存.