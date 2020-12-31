---
title: 悲观锁与乐观锁
date: 2020-12-22 22:30:10
tags: [锁]
---

## 定义

### 悲观锁
认为保护的的数据是及其不安全的，随时都有可能被人修改，这个是时候通过添加悲观锁，使得在解锁之前这部分数据一直属于当前线程，其他线程想要访问这部分数据必须等我释放锁。


### 实现方法

* `Mysql` 排他锁

```
update table set stock = stock -1 where id = xxx for update;
```

* 对代码块加锁，比如PHP中文件锁，或者是Go中的sync.Mutex



## 乐观锁
乐观锁是一种思想，一般情况下我们乐观地认为我们接下来的操作不会存在数据冲突，只有在进行提交更新的时候，才会对数据的冲突与否进行检测。


### 通过版本号实现

`Mysql中`，我们一般是通过在表结构中维护一个版本号(version)来实现乐观锁：

```
update table set stock = stock - 1, version = version + 1 where id xxx and version = 1
```

注意： 此方法会存在`ABA`问题


### Redis `watch`加`事务`实现

具体实现如下:

```
<?php  
$redis = new redis();  
$result = $redis->connect('10.10.10.119', 6379);

//产品销量
$mywatchkey = $redis->get(sales);  

//秒杀库存 
$stock = 100;

if ($mywatchkey >= $stock) {
    exit("秒杀结束");
}

//监控销量
$redis->watch('sales'); 

//开启事务
$redis->multi();

$redis->incr('sales');  //将 key 中储存的数字值增一 ,如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。

$res=$redis->exec(); //成功1 失败0

if ($res) {
    //扣减库存
    include 'db.php';
    $sql="update products set store=store-1 where id=1";
    if($mod->exec($sql)){
      echo "秒杀成功";
    } else{
        //销量减1
        $redis->decr('sales');
        echo "秒杀失败";
    }
} else {
     exit('秒杀失败');
}

```


## 总结：

### 悲观 VS 乐观锁

* 用户体验上，乐观锁更好，因为悲观锁存在阻塞
* 数据层，悲观锁能保证数据的强一致性，而乐观锁有可能只能保证数据的最终一致性，这和乐观锁锁的具体实现相关。
* 乐观锁本身是不加锁的，只是在更新提交时判断一下数据是否被其他线程更新了
* 锁的选择需要我们结合实际的业务场景来做取舍






