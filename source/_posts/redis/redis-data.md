---
title: Redis设计与实现
date: 2022-07-25 10:03:41
tags: Redis
categories: Redis
---

## 数据结构

### 字符串

在redis中，所有的键一定是字符串。redis虽然是用C语言编写的，但是没有直接使用C语言风格字符串（C-style String），而是实现了一种名为**简单动态字符串**（simple dynamic string, SDS）作为字符串的默认实现。C-style String只会用于**字符串字面量**。

```c
struct sdshdr{
  // 记录字符串长度
  int len;
  // 记录buf中未使用的字节数
  int free;
  char buf[];
}
```

SDS遵循C-style String的风格，在末尾自动添加`\0`，也会自动申请额外1字节的空间，这些操作都是SDS自动维护，对用户是透明的，这也使得SDS可以直接使用C语言中的字符串操作函数。

为什么SDS比C-style String更适合redis：

1. 常数复杂度获取字符串长度
2. 杜绝内存溢出，预检查buf空间
3. 减少修改字符串时带来的内存重新分配次数。小于1MB时预分配和len一样的free空间；超过1MB时每次多分配1MB额外空间。
4. C-style String必须符合某种编码，并且只能在末尾出现空字符，使得其只能保存文本数据，不能存储二进制文件。
5. 可以重用部分`string.h`中的函数

### 链表

```C
typedef struct listNode {
  struct listNode *prev;
  struct listNode *next;
  void *value;
}listNode
typedef struct list {
  listNode *head;
  listNode *tail;
  unsigned long len;
  // 节点值复制函数
  void *(*dup) (void *ptr);
  void (*free) (void *ptr);
  int (*match) (void *ptr, void *key);
}list
```

### 字典

redis的字典基于哈希表实现。

```C
typedef struct dictht {
  // 哈希表数组的指针
	dictEntry **table;
  unsigned long sie;
  // 哈希表大小掩码，用于计算索引值，总是等于size-1
  unsigned long sizemask;
  unsigned long used;
} dictht:

typedef struct dictEntry {
  void *key;
 	union{
    void *val;
    uint64_t u64;
    int64_t s64;
  } v;
  // 指向下一个哈希表节点，形成链表
  struct dictEntry *next;
} dictEntry;

typedef struct dict {
  dictType *type;
  void *privdadta;
  dictht ht[2];
  int rehashidx;
} dict;

typedef struct dictType {
  unsigned int (*hashFunction)(const void *key);
  void *(*keyDup)(void *privdata, const void *key);
  void *(*valDup)(void *privdata, const void *obj);
  int (*keyCompare)(void *privdata, const void *key1, const void *key2);
  void (*keyDestructor)(void *privdata, void *key);
  void (*valDestructor)(void *privdata, void *obj);
} dictType;
```

`ht[0]`和`ht[1]`都是哈希表，一般情况下，字典只使用 `ht[0]`，只有在需要进行rehash的时候才会使用`ht[1]`。当不进行rehash的时候，`rehashidx`始终为-1。当负载因子不在合理范围时，需要rehash来调整表大小：

1. 给`ht[1]`分配空间，如果是扩展则为`ht[0].used*2`的下一个2^n^,  如果是收缩则为`ht[0].used`的下一个2^n^
2. 将所有的`ht[0]`rehash到`ht[1]`上
3. 全部迁移之后，释放`ht[0]`，将`ht[1]`设置为`ht[0]`，为`ht[1]`创建一个新的空表。

如果服务器没有正在执行BGSAVE或者BGREWRITEAOF命令，name负载因子大于等于1时触发。如果正在执行，name大于等于5时触发。考量是在执行以上2种命令时，redis会创建子进程，根据**写时复制**的优化机制，需要尽可能减少对服务器内存IO的压力，因此调大负载因子的阈值。

注意，rehash是渐进式的，不是一口气完成的。因为如果哈希表很大，服务器需要马上处理完的话可能负担过重，因此是每次对字典执行操作时顺带做一点，每搬运一个Entry就将rehashidx增加1，全部完成后置为-1。在rehash期间，redis会现在`ht[0]`上查找数据，找不到则在`ht[1]`上再找。另外，插入操作会在`ht[1]`上进行，包装`ht[0]`上的数据只减不增。                                                                         

type属性和privdata属性是针对不同类型的键值对，为创建多态字典而设置的。每个dictType结构保存了一组用于操作特定类型键值对的函数。

redis的哈希算法使用MurmurHash2。

由于dictEntry组成的链表没有尾部节点的指针，所以遇到冲突时，新节点会插入表头位置。

### 跳跃表

有序数据结构，每个阶段维持多个其他节点的指针，支持平均O(logN)，最坏O(N)的节点查找。

在redis中跳跃表用的不多，只用于实现有序集合键和在集群节点中作为内部数据结构。

### 整数集合

当一个集合只包含整数，且数量不多时，就会使用整数集合作为底层实现。`contents`中的数字从小到大有序，没有重复。

```C
typedef struct  intset {
  // 编码方式
	uint32_t encoding;
  // 元素数量
	uint32_t length;
  // 保存元素的数组
	int8_t contents[];
}
```

虽然`contents`被声明为`int8_t`，但是真正的类型取决于`encoding`如何规定每个整数的位数。

当新元素的类型比整数集合内所有元素都长时，整数集合需要**升级**：根据新元素的类型，扩展`contents`的大小，重新分配空间；将原有的元素转换成新类型后，**从后往前**重新添加到`contents`。

注意：引发升级的新元素的长度总是比整数集合内所有现有元素长，所以这个新元素要么比现有元素都小，要么都大。因此插入位置只会在`contents`的开头或者末尾。

1. 升级机制提升了整数集合的灵活性，为C语言这种静态语言提供了用一个结构保存不同类型整数对的方法。
2. 节约了内存。

注意：**不支持降级**操作。

### 压缩列表

列表键和哈希键的底层实现之一。当列表键只包含少量列表项，并且列表项是小整数或短字符串时，就会采用压缩列表。

压缩列表在内存中由几个连续的组成部分：

1. zlbytes：记录压缩列表占用字节数，在重新分配内存和计算zlend时使用
2. zltail，记录尾节点距离起始位置的偏移量，可以直接确定尾节点的位置
3. zllen：记录节点数量。当这个值等于UINT16_MAX时，说明达到上限，真实数量需要遍历后获得
4. entryX：列表节点，长度不定
5. zlend：特殊值0xFF，标记压缩列表的末端

列表节点：

1. previous_entry_length：1字节或5字节，取决于前一个节点的字节数是否超过254。有这个就可以得知前一个节点的起始位置，从而实现列表的倒序遍历。
2. encoding：记录了content保存数据类型和长度。要么是1、2、5字节长度，以00、01、10开头的字节数组编码（用来表示不同的长度），要么是1字节长度，以11开头的整数编码。
3. content

连锁更新：压缩列表内恰好有多个连续的，长度介意250到253字节的节点，在前面新增节点会导致它们的previous_entry_length不够用1字节，需要扩容，从而导致后续节点连锁反应。

### 对象

redis没有用以上数据结构直接实现键值对数据库，而是基于这些数据结构创建了对象系统，包括字符串对象、列表对象、哈希对象、集合对象和有序集合对象。还基于**引用计数**实现**内存回收**机制和**对象共享**。redis对象还带有访问时间记录，在服务器启用maxmemory时，空转较久的键可能被优先删除。

```C
typedef struct redisObject {
  // 类型，5个常量
	unsigned type:4;
  unsigned encoding:4;
  // 指向底层实现数据结构的指针
  void *ptr;
  // ...
} orbj;
```

[<img src="https://s1.ax1x.com/2022/07/26/jzkNDO.png" alt="jzkNDO.png" style="zoom:50%;" />](https://imgtu.com/i/jzkNDO)

encoding有以上这些类型。每种对象至少基于2中底层实现，见下表：

[<img src="https://s1.ax1x.com/2022/07/26/jzASR1.png" alt="jzASR1.png" style="zoom:50%;" />](https://imgtu.com/i/jzASR1)

#### 字符串对象

字符串对象的编码可以是：

1. `int`：如果字符串是整数值并且可以用`long`类型表示，`ptr`属性会保存整数值，`void*`会转成`long`。
2. `raw`：如果是字符串值，且长度大于32字节
3. `embstr`：如果是字符串值，且长度≤32字节。这是专门用于保存短字符串的一种优化编码方式。虽然和`raw`编码一样，需要用`redisObject`和`sdshdr`两个结构来表达字符串，但是`raw`会调用两次内存分配函数，`embstr`只会调用一次，分配一块连续内存给这2个结构。这样释放内存时也只用调用一次，而且2个结构再内存中紧挨着可以利用缓存优势。

对于`int`编码的字符串，如果对这个“整数”进行了属于字符串的操作，则会改变编码为`raw`。

redis只实现了对`int`和`raw`的修改，因此`embstr`实际上是只读的，对其的修改都会令编码改变为`raw`。

#### 列表对象

当同时满足以下条件时，列表对象使用`ziplist`编码：

1. 保存的所有字符串元素的长度都小于64字节
2. 元素数量少于512个

否则使用`linkedlist`编码。

注意：这两个上限值都可以在配置文件中修改。

#### 哈希对象

底层是`ziplist`或者`hashtable`。

使用压缩列表时，每次插入键值对都是紧挨着插入队尾。

当同时满足以下条件时，哈希对象使用`ziplist`编码：

1. 保存的所有键和值的字符串元素的长度都小于64字节
2. 元素数量少于512个

否则使用`hashtable`编码。

#### 集合对象

底层是`intset`或者`hashtable`。

使用`intset`时，要求所有元素为整数，且不超过512个。

当使用哈希表时，只使用键，值全部为NULL。

#### 有序集合对象

底层是`ziplist`或者`skiplist`。

使用`ziplist`时，每个元素使用2个紧挨的压缩列表节点来保存，第一个保存元素的成员，的第二个保存元素的分值。

使用`skiplist`时，一个`zset`结构同时包含一个字典和一个跳跃表。跳跃表节点的object属性保存元素的成员，score属性保存分数。通过跳跃表，可以对有序集合进行**范围性操作**。字典创建了成员到分值的映射，可以实现O(1)的分值查询。

字典和跳跃表在底层是共用元素的成员和分值的，不会产生浪费。

当元素数量小于128个，所有元素成员的长度小于64字节时，使用`ziplist`编码。

#### 类型检查与命令多态

Redis中有些命令可以对任意类型执行，但是有些只能用于特定类型，说明其中有类型检查的操作。类型检查通过`redisObject`的`type`属性实现的。

同种对象有不同的底层实现，命令需要根据底层实现的不同，或者是作用的键类型不同，选择正确的API来执行，这就是**多态命令**。

#### 内存回收

```C
typedef struct redisObject {
  // ...
  int refcount;
} orbj;
```

C语言本身没有内存回收功能，因此Redis自己实现了基于引用计数的内存回收机制，由`redisObject`的`refcount`属性来记录。

#### 对象共享

引用计数机制允许实现对象共享，有利于节约内存。Redis在初始化服务器时会创建0到9999共一万个字符串对象，因为它们大概率会被用来共享。

用`object refcount`命令可以查看引用计数。

注意：只支持整数值的共享，字符串对象和包含多个值的对象验证相同的复杂度太高。

#### 空转时长

```C
typedef struct redisObject {
  // ...
  unsigned lru:22;
} orbj;
```

该属性记录了对象最后一次被命令程序访问的时间。`object idletime`可以查看给定键的空转时长。这个命令实现是特殊的，在访问键的值对象时，不会修改值对象的lru属性。

如果服务器开了maxmemory选项，并且服务器的回收内存算法为`volatile-lru`或者`allkeys-lru`，那内存超过上限时优先释放空转时长较高的键。



## 单机数据库

```C
struct redisServer {
	// ...
	// 保存服务器中所有数据库的数组
	redisDb *db;
	// 服务器中数据库数量
	int dbnum;
	// ...
}
```

Redis中所有服务器都要保存在服务器状态`redis.h/redisServer`结构对的db数组中，类型为`redis.h/redisDb`。`dbnum`默认为16。

```C
struct redisClient {
	// ...
	// 记录客户端当前正在使用的数据库
	redisDb *db;
};
```

Redis是键值对数据库，每个数据库的所有键值对都保存在**键空间**里：

```C
struct redisDb {
	// ...
	// 数据库键空间，保存数据库中所有的键值对
	dict *dict;
};
```

对数据库的读写还会附带一些维护操作：

1. 更新服务器键空间的命中或不命中次数
2. 更新LRU
3. 如果发现键以过期，要删除键后再进行剩余操作
4. 如果有使用WITCH命令监视该键，则修改后要将键标记为dirty，从而让事务注意到这个键已被修改
5. 服务器每修改一个劲键，都会对脏键计数器加1，这个计数器会触发服务器的持久化和复制操作
6. 如果服务器开启了数据库通知功能，那么在对键修改后要按照配置发送相应的数据库通知。

`expire`命令可以设置某个键的生存时间，在多少秒后删除。

`expireat`命令可以设置某个键的过期时间，在某个时间戳来临时删除。

`ttl`命令查看一个键的生存时间或者过期时间。

以上单位都是秒。`pexpire`、`pexpireat`、`pttl`以毫秒为单位。

以上的设置命令，最后都会经过转换，然后执行`pexpireat`。

```C
struct redisDb {
	// ...
	// 过期字典，保存键的过期时间
	dict *expires;
};
```

过期时间保存在`redisDb`中的过期字典中。虽然过期字典和键空间内都保存了键，但是指向的键对象是共享的，所以没有浪费内存。

`persist`命令可以移除一个键的过期时间。

检查一个键是否过期，有一个函数，大致操作是获取过期时间，和当前UNIX时间戳比较。另一种方法是用`ttl`或`pttl`命令查看过期时间是否大于0，但是实际上Redis还是使用与函数操作类似的操作，因为直接访问过期字典比执行一个命令要快。

#### 过期删除策略

