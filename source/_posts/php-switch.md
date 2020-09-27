---
title: siwtch 的非常规写法
date: 2020-09-27 15:44:10
tags: [PHP]
---

PHP中Switch的非常规写法

## Switch 语法

`switch` 语句用于根据多个不同条件执行不同动作。
工作原理：首先对一个简单的表达式 n（通常是变量）进行一次计算。将表达式的值与结构中每个 case 的值进行比较。如果存在匹配，则执行与 case 关联的代码。代码执行后，使用 break 来阻止代码跳入下一个 case 中继续执行。default 语句用于不存在匹配（即没有 case 为真）时执行。

## 常规写法

```
<?php
$favcolor="red";
switch ($favcolor)
{
case "red":
    echo "你喜欢的颜色是红色!";
    break;
case "blue":
    echo "你喜欢的颜色是蓝色!";
    break;
case "green":
    echo "你喜欢的颜色是绿色!";
    break;
default:
    echo "你喜欢的颜色不是 红, 蓝, 或绿色!";
}

//output: 你喜欢的颜色是红色!
?>
```

## 非常规写法

```
<?php
$score = 99;
switch (true) {
case $score > 95:
    echo "Perfect!";
    break;
case $score > 60:
    # code...
    echo "You can do better";
    break;
default:
    echo "Oh, seems a little bad!";
    break;
}

//output: Perfect!
?>

```

上面的两种写法，相信更多的是第一种写法。


首先我们先来了解一个专业术语 `表达式`, 这个词经常出现在各种语言的手册里面。让我们来看一下PHP官方给出的解释是啥?

表达式是 PHP 最重要的基石。在 PHP 中，几乎所写的任何东西都是一个表达式。简单但却最精确的定义一个表达式的方式就是“任何有值的东西”。

这时你再回过头来看上面的两个例子， $favcolor 和 true 其实都是一个表达式, 第一种写法switch中的表达式值为red, 所以他匹配到了 case "red",
同理第二种写法switch中的表达式值为true, 刚好第一个 case $score > 95 值为true与之匹配。