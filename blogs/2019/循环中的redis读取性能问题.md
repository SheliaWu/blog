---
title: 循环中redis读取性能问题
date: 2019-04-18
tags: redis 循环
---
> 循环中进行读取操作是一种非常耗内存的方式

公司的业务中经常使用redis来存储数据，所以操作redis是我日常工作的一部分。之所以选择redis，老大解释的原因有三个：一是因为redis读取容易和高速，二是redis支持丰富的数据类型，三是redis的操作都是原子性的，要么成功要么失败。选择redis的原因也是redis相对于其他数据库的优势。部分业务会根据需要持久化，落库。

今天遇到一个关于redis读取性能的问题，代码大概如下：
```php
$recrods =  $redis->hgetall("records");
foreach($records as $key => $value){
    $value1 =  $redis->hget("key1",$key);
    $value2 =  $redis->hget("key2",$key);
    $value3 =  $redis->hget("key3",$key);
    //...省略其他代码
}
```
首先获取某个key下的全部键值，遍历这个键值表又操作了几次读取操作。当业务量少的时候，没有发现什么问题，当并发请求变多的时候，突然发现服务器的cpu飙升报警，redis进程被卡住了。经过排查，发现是上面这段循环代码导致的。

回头看这段代码确实写得很有问题，想当然的什么都使用循环解决，却没有考虑到程序的性能问题。

优化这段代码，首先想到的是通过管道来进行读取操作，但是发现优化不是很明显。接着想到问题显然出现在循环中多次进行读取操作，多次请求了redis导致的，如果可以把多个请求合并成一个请求就好了。

redis的`hmget`命令适合用于这个场景下的问题，此命令用于返回哈希表中，一个或多个给定字段的值。所以把所有的field合并成一个数组，就可以返回对应的多个值，然后再进行处理，就不用再一个一个进行`hget`操作。
优化后的代码大概如下：
```php
$recrods =  $redis->hgetall("records");
foreach($records as $key => $value){
    $keys[] = $key;
}
$values1 =  $redis->hmget("key1",$keys);
$values2 =  $redis->hmget("key2",$keys);
$values3 =  $redis->hmget("key3",$keys);
//...省略其他代码
```

可以有多个新建的数组，数组循环是一件很快的事情，但是在循环中多次请求存取并不是一件美好的事情。重构后的代码，虽然牺牲了空间，但节省了时间。
