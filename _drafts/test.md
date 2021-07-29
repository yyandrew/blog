○ Redis是我在大约3年前为了解决一个实际问题而创造出来的;简单的不说，当时我在尝试做一个使用硬盘存储关系数据库(on-disk SQL database)无法完成的事情--在一台我能够支付得起的小虚拟机上面处理大量写入负载。

我要解决的问题在概念上并不复杂：多个网站会通过一个小型的JavaScript追踪器发送页面访问记录(page view)，而我的服务器需要为每个网站保存一定数量的最新页面访问记录，并通过页面将这些记录实时的展示给用户观看。

在最大负载达到每秒数千条页面记录的情况下，无论我如何进行优化，我所使用的关系数据库都没有办法在这个小虚拟机上处理如此大的负载。因为囊中羞涩，我没有办法对虚拟机进行升级，并且我觉得应该有更简单的方法来处理一个由推入值组成的列表。最终，我决定自己写一个实验性质的内在数据库原型，这个数据库使用列表作为基本数据类型，并且能够对列表的两端执行常数时间复杂度的弹出(popup)和推入(push)操作。长说短说吧，这个内在数据库的想法的确奏效了，于是我用C语言重写了最初的数据库原型，并给它加上了基于子进程实现的持久化特性，Redis就这样诞生了。
☆ 根据作者的经历可以看出Redis天然具有节省硬件资源，读写快速，功能强大的特性

○ Redis可以存储键与5种不同数据结构类型之间的映射。这5种数据结构类型分别为STRING(字符串)、LIST(列表)、SET(集合)、HASH(散列)和ZSET(有序集合)。有一部分Redis命令对这5种结构都是通用的，如DEL，RENAME，TYPE等；但也有一部分Redis命令只能对特定的一种或者两种结构使用。
○ 字符串(STRING)

定义：

```c
struct sdshdr {
 long len;
 long free;
 char buf[];
};
```

\* buf数组：字符串的实体，保存字符串的内容

\* len字段：记录数组大小

\* free字段：记录buf数组还有多少空间

```shell
127.0.0.1:6379>set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
```

○ 列表(LIST)结构可以有序地存储多个字符串。

```shell
127.0.0.1:6379> lpush mylist1 value1
(integer) 1
127.0.0.1:6379> llen mylist1
(integer) 1
127.0.0.1:6379> lpop mylist1
"value1"
127.0.0.1:6379> lpush mylist1 value1
(integer) 1
127.0.0.1:6379> lpush mylist1 value2
(integer) 2
127.0.0.1:6379> lpush mylist1 value3
(integer) 3
127.0.0.1:6379> lrange mylist1 0 2
1) "value3"
2) "value2"
3) "value1"
```

○ 集合(SET)和列表都可以存储多个字符串，它们之间的不同在于，列表可以存储多个相同的字符串，而集合则通过使用散列表来保证自己存储的每个字符串都是各不相同的。

```shell
127.0.0.1:6379> sadd myset1 member1
(integer) 1
127.0.0.1:6379> sadd myset1 member2
(integer) 1
127.0.0.1:6379> scard myset1
(integer) 2
127.0.0.1:6379> sismember myset1 member1
(integer) 1
127.0.0.1:6379> sismember myset1 member5
(integer) 0
127.0.0.1:6379> smembers myset1
1) "member2"
2) "member1"
```



○ 散列(HASH)可以存储多个键值对之间的映射。

```shell
127.0.0.1:6379> hset myhash name andrew
(integer) 1
127.0.0.1:6379> hset myhash sex male
(integer) 1
127.0.0.1:6379> hset myhash age 33
(integer) 1
127.0.0.1:6379> hget myhash name
"andrew"
127.0.0.1:6379> hkeys myhash
1) "name"
2) "sex"
3) "age"
127.0.0.1:6379> hexists myhash name
(integer) 1
```

○ 有序集合(ZSET)和散列(HASH)一样，都用于存储键值对：有序集合的键被称为成员（member），每个成员都是独一无二的；它的值被称为分值（score)，分值必须为浮点数。有序集合是Redis里唯一一个既可以根据成员访问元素（这一点和散列一样），又可以根据分值以及分值的排序来访问元素的结构。

```shell
127.0.0.1:6379> zadd zset-key 728 member0
(integer) 1
127.0.0.1:6379> zadd zset-key 982 member0
(integer) 0
127.0.0.1:6379> zadd zset-key 982 member1
(integer) 1
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1.  "member0"
2.  "982"
3.  "member1"
4.  "982"
127.0.0.1:6379> zrem zset-key member0
(integer) 1
127.0.0.1:6379> zrem zset-key member0
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member1"
2) "982"
```


## Redis排序命令

Redis支持对列表、集合、有序集合类型进行排序，

sort命令完整格式如下：`SORT key [By pattern] [LIMIT start count] [GET pattern] [ASC|DESC] [ALPHA][STORE dstkey]`

* SORT

  ```shell
  127.0.0.1:6379> lpush mylist 2
  (integer) 1
  127.0.0.1:6379> lpush mylist 1
  (integer) 2
  127.0.0.1:6379> lpush mylist 3
  (integer) 3
  127.0.0.1:6379> sort mylist
  1) "1"
  2) "2"
  3) "3"
  ```

* [ASC | DESC] [ALPHA]

  SORT命令默认排序方式是从小到大(ASC)

  ```shell
  127.0.0.1:6379> lpush namelist Tom
  (integer) 1
  127.0.0.1:6379> lpush namelist Jack
  (integer) 2
  127.0.0.1:6379> lpush namelist Mick
  (integer) 3
  127.0.0.1:6379> lpush namelist Wade
  (integer) 4
  127.0.0.1:6379> sort namelist alpha
  1) "Jack"
  2) "Mick"
  3) "Tom"
  4) "Wade"
  127.0.0.1:6379> sort namelist desc alpha
  1) "Wade"
  2) "Tom"
  3) "Mick"
  4) "Jack"
  ```

* [BY pattern]

  ```shell 
  127.0.0.1:6379> set name1 102
  OK
  127.0.0.1:6379> set name2 103
  OK
  127.0.0.1:6379> set name3 101
  OK
  127.0.0.1:6379> sort mylist by name*
  1) "3"
  2) "1"
  3) "2"
  ```

  模式`name*`代表使用mylist集合中元素的值填充`*`,按照name1, name2, name3这3个key对应的排序，返回的是排序后mylist集合中的元素。

* [LIMIT start count]

  使用LIMIT选项能够限定返回结果的数量。

  ```shell
  127.0.0.1:6379> sort mylist limit 0 2
  1) "1"
  2) "2"
  ```

* [STORE dstkey]

  把排序结果缓存起来可以减少CPU的开销。使用STORE选项将排序结果可在到指定的key中。

  ```
  127.0.0.1:6379> sort mylist by name* store cachelist
  (integer) 3
  127.0.0.1:6379> type cachelist
  list
  127.0.0.1:6379> lrange cachelist 0 -1
  1) "3"
  2) "1"
  3) "2"
  ```

  

## 事务处理

一般情况下，Redis接收到一个客户端连接发来的命令，立刻执行并返回结果。但是当连接发出multi命令时，此连接便进入一个事务上下文，Redis把引连接发来的命令存入一个队列， 当此连接发出exec命令，并将所有命令执行的结果打包一起返回给客户端连接，然后此连接便结束事务上下文。

```shell 
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set num 1
QUEUED
127.0.0.1:6379> incr num
QUEUED
127.0.0.1:6379> exec
1) OK
2) (integer) 2
```



从上面的例子看到`set num 1`和`incr num`命令发出之后并没有立即执行。而是存放到队列中。调用`exec`命令，这两个命令才开始连续执行，最后返回两个命令执行后的结果。调用`discard`伪命令取消一个事务。

## 持久化

Redis是基于内在的数据库，内在数据库有个严重的弊端：突然宕机或者断电时，内在的数据不会保存。为了解决这个问题，Redis提供两种持久化方式：内存快照和日志追加。

* 内存快照

  内存快照方式是将内存中的数据以快照方式写入二进制文件中，默认名为dump.rdb。

  客户端使用`save`命令告诉Redis需要做一次内存快照操作。save命令在主线程中保存内存快照，Redis由单线程处理所有请求，执行`save`命令可能阻塞其它客户端。

  另外要注意，内存快照每次都把内存数据完整地写入硬盘，而不是只写入增量数据。所以如果数据量大，定稿操作比较频繁，从而严重影响性能。

  内存快照相关的配置选项如下：

  ```shell
  save <seconds> <changes>
  ```

  

  比如：`save 900 1`表示经过900秒或者数据一更改一次就进行内存快照操作。

* 日志追加

  日志追加(aof)方式是把增加、修改数据的命令通过wirte函数追加到文件尾部(默认是appendonly.aof)。Redis重启时读取appendonly.aof文件中的所有命令并且执行，从而把数据写入内存。

  ```shell
  # appendfsync always # 每次收到增加或者修改命令就会立刻强制定稿磁盘
  appendfsync everysec # 每秒强制写入磁盘
  # appendfsync no # 是否写入磁盘完全依赖操作系统
  ```

  

## 主从同步

主从同步可以防止主机崩溃导致网站不能正常运作。

Redis主从同步设置很简单，设置好Slave服务器之后，Slave自动和Master建立连接，发送SYNC命令。

![img](file:///private/var/folders/d7/lh4t_zzd1pxcbxtp3j5ydwp00000gn/T/WizNote/62463b33-dbfb-4e1d-815e-58c8f097ca70/index_files/2a166407-7cf8-4136-af9b-eda4dc33209d.jpg)

○Redis是一个可以用来解决问题的工具，它既拥有其它数据库不具备的数据结构，有拥有内存存储（这使得Redis的速度非常快）、远程（这使得Redis可以与多个客户端和服务器进行连接）、持久化（这使得服务器在重启之后仍然保持重启之前的数据）和可扩展（通过主从复制和分片）等多个特征。
