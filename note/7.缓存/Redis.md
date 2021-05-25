

# 一些概念

## 相关知识

| 端口6379从何而来  Alessia  Merz                                                                          ![](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210516214236.png) | 默认16个数据库，类似数组下标从0开始，初始默认使用0号库  使用命令 select  <dbid>来切换数据库。如: select 8   统一密码管理，所有库同样密码。  dbsize查看当前数据库的key的数量  flushdb清空当前库  flushall通杀全部库 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |

## Redis是单线程+多路IO复用技术

多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）

串行  vs  多线程+锁（memcached） vs  单线程+多路IO复用(Redis)

（与Memcache三点不同: 支持多数据类型，支持持久化，单线程+多路IO复用） 

 ![image-20210516214217642](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210516214217.png)

# 数据类型

[Redis数据类型命令查询](http://www.redis.cn/commands.html)

## 通用

```properties
keys *		查看当前库所有key    key数量大时慎用
exists key	判断某个key是否存在
type key 	查看你的key是什么类型
del key     删除指定的key数据
unlink key 	根据value选择非阻塞删除,仅将keys从key keys space元数据中删除，真正的删除会在后续异步操作。
expire key 10   10秒钟：为给定的key设置过期时间
ttl key 	查看还有多少秒过期，-1表示永不过期，-2表示已过期

select		命令切换数据库
dbsize		查看当前数据库的key的数量
flushdb		清空当前库
flushall 	通杀全部库
```

## String

- String是Redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

- String类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

- String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M
- incr 与 decr / setnx 等是原子操作

### 命令

```properties
set   <key><value>		添加键值对
	*NX：当数据库中key不存在时，可以将key-value添加数据库
	*XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥
	*EX：key的超时秒数
	*PX：key的超时毫秒数，与EX互斥

get   <key>						查询对应键值
append  <key><value>			将给定的<value> 追加到原值的末尾
strlen  <key>					获得值的长度
setnx  <key><value>				只有在 key 不存在时    设置 key 的值
incr  <key>						将 key 中储存的数字值增1,只能对数字值操作，如果为空，新增值为1
decr  <key>						将 key 中储存的数字值减1,只能对数字值操作，如果为空，新增值为-1
incrby / decrby  <key> <步长>		将 key 中储存的数字值增减。自定义步长。

getrange  <key><起始位置><结束位置>	 获得值的范围，类似java中的substring，前包，后包
setrange  <key><起始位置><value>	用 <value>覆写<key>所储存的字符串值，从<起始位置>开始(索引从0开始)。
setex  <key><过期时间><value>		设置键值的同时，设置过期时间，单位秒。
getset <key><value>				  以新换旧，设置了新值同时获得旧值。

```

### 数据结构

**String的数据结构为简单动态字符串(Simple Dynamic String,缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.**

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210516215053.png" alt="image-20210516215053471" style="zoom:150%;" />

如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，**如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。**

## List列表

- 单键多值

- Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

- 它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210516215224.png" alt="image-20210516215224873" style="zoom:150%;" />

### 命令

```properties

lpush/rpush  <key><value1><value2><value3> 	.... 从左边/右边插入一个或多个值。
lpop/rpop  <key>				从左边/右边吐出一个值。值在键在，值光键亡。
rpoplpush  <key1><key2>从<key1>	列表右边吐出一个值，插到<key2>列表左边。

lrange <key><start><stop>		按照索引下标获得元素(从左到右)
lrange mylist 0 -1   			0左边第一个，-1右边第一个，（0-1表示获取所有）
lindex <key><index>				按照索引下标获得元素(从左到右)
llen <key>						获得列表长度 
linsert <key>  before/after <value><newvalue>		在<value>的前面/后面插入<newvalue>插入值
lrem <key><n><value>						从左边删除n个value(从左到右)
lset<key><index><value>						将列表key下标为index的值替换成value

```

### 数据结构

**List的数据结构为快速链表quickList。**

**首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。**

它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

**当数据量比较多的时候才会改成quicklist。**

因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210516215406.png" alt="image-20210516215406732" style="zoom:150%;" />

**Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。**

## Set集合

Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以**自动排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

**Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的复杂度都是O(1)。**

一个算法，随着数据的增加，执行时间的长短，如果是O(1)，数据增加，查找数据的时间不变

### 命令

```properties

sadd <key><value1><value2> 	.....	将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
smembers <key>						取出该集合的所有值。
sismember <key><value>				判断集合<key>是否为含有该<value>值，有1，没有0
scard<key>							返回该集合的元素个数。
srem <key><value1><value2> .... 	删除集合中的某个元素。
spop <key>							随机从该集合中吐出一个值。
srandmember <key><n>				随机从该集合中取出n个值。不会从集合中删除 。
smove <source><destination> value	把集合中一个值从一个集合移动到另一个集合
sinter <key1><key2>					返回两个集合的交集元素。
sunion <key1><key2>					返回两个集合的并集元素。
sdiff <key1><key2>					返回两个集合的差集元素(key1中的，不包含key2中的)

```

### 数据结构

- Set数据结构是dict字典，字典是用哈希表实现的。

- Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。



## Hash

- Redis hash 是一个键值对集合。

- Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

- 类似Java里面的Map<String,Object>

- 用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储

主要有以下2种存储方式：

 

| 每次修改用户的某个属性需要，先反序列化改好后再序列化回去。开销较大。 |
| ------------------------------------------------------------ |
| ![image-20210521230934135](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521230934.png) |
| 用户ID数据冗余                                               |
| ![image-20210521230942133](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521230942.png) |
| **通过 key(用户ID) + field(属性标签) 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题** |
| ![image-20210521230948649](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521230948.png) |

### **命令**

```properties
hset <key><field><value>		给<key>集合中的  <field>键赋值<value>
hget <key1><field>从<key1>		集合<field>取出 value 
hmset <key1><field1><value1><field2><value2>		... 批量设置hash的值
hexists<key1><field>					查看哈希表 key 中，给定域 field 是否存在。 
hkeys <key>								列出该hash集合的所有field
hvals <key>								列出该hash集合的所有value
hincrby <key><field><increment>			为哈希表 key 中的域 field 的值加上增量 1   -1
hsetnx <key><field><value>				将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .

```

### **数据结构**

Hash类型对应的数据结构是两种：**ziplist（压缩列表），hashtable（哈希表）**。

**当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。**

## ZSet 有序集合

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分（score**）,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

### **命令**

```properties
zadd  <key><score1><value1><score2><value2>		将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
zrange <key><start><stop>  [WITHSCORES]   		返回有序集 key 中，下标在<start><stop>之间的元素带WITHSCORES，可以让分数一起和值返回到结果集。
zrangebyscore key minmax [withscores] [limit offset count]
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 
zrevrangebyscore key maxmin [withscores] [limit offset count]       同上，改为从大到小排列。 
zincrby <key><increment><value>      								为元素的score加上增量
zrem  <key><value>													删除该集合下，指定值的元素 
zcount <key><min><max>												统计该集合，分数区间内的元素个数 
zrank <key><value>													返回该值在集合中的排名，从0开始。

```

### 数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

###  **跳跃表（跳表）**

1、简介

  有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

2、实例

  对比有序链表和跳跃表，从链表中查询出51

（1）  有序链表

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231447.png" alt="image-20210521231447586" style="zoom:150%;" />

要查找值为51的元素，需要从第一个元素开始依次查找、比较才能找到。共需要6次比较。

（2）  跳跃表

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231508.png" alt="image-20210521231508452" style="zoom:150%;" />

从第2层开始，1节点比51节点小，向后比较。

21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层

在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下

在第0层，51节点为要查找的节点，节点被找到，共查找4次。

 

从此可以看出跳跃表比有序链表效率要高



# 新数据类型

## bitmaps

Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

（1）  Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。

（2）  Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做偏移量。

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233148.png" alt="image-20210521233148712" style="zoom:150%;" />

### 命令

#### setbit

（1）格式

`setbit<key><offset><value>`设置Bitmaps中某个偏移量的值（0或1）                           

*offset:偏移量从0开始

 ![image-20210521233258981](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233259.png)

（2）实例

每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用偏移量作为用户的id。

设置键的第offset个位的值（从0算起） ， 假设现在有20个用户，userid=1， 6， 11， 15， 19的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图

 ![image-20210521233305571](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233305.png)

unique:users:20201106代表2020-11-06这天的独立访问用户的Bitmaps

 ![image-20210521233309573](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233309.png)

注：

很多应用的用户id以一个指定数字（例如10000） 开头， 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费， 通常的做法是每次做setbit操作时将用户id减去这个指定数字。

在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞。

 

#### getbit

（1）格式

`getbit<key><offset>`获取Bitmaps中某个偏移量的值

 ![image-20210521233317544](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233317.png)

获取键的第offset位的值（从0开始算）

 

（2）实例

获取id=8的用户是否在2020-11-06这天访问过， 返回0说明没有访问过：

 ![image-20210521233325011](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233325.png)

 

注：因为100根本不存在，所以也是返回0

 

#### bitcount

统计**字符串**被设置为1的bit数。一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数的设置，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位，start、end 是指bit组的字节的下标数，二者皆包含。

（1）格式

`bitcount<key>[start end]` 统计字符串从start字节到end字节比特值为1的数量

 ![image-20210521233331770](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233331.png)

 

（2）实例

计算2022-11-06这天的独立访问用户数量

 ![image-20210521233340257](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233340.png)

 

start和end代表起始和结束字节数， 下面操作计算用户id在第1个字节到第3个字节之间的独立访问用户数， 对应的用户id是11， 15， 19。

 ![image-20210521233343896](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233343.png)

 

举例： K1 【01000001 01000000 00000000 00100001】，对应【0，1，2，3】

bitcount K1 1 2 ： 统计下标1、2字节组中bit=1的个数，即01000000 00000000

--》bitcount K1 1 2 　　--》1

 

bitcount K1 1 3 ： 统计下标1、2字节组中bit=1的个数，即01000000 00000000 00100001

--》bitcount K1 1 3　　--》3

 

bitcount K1 0 -2 ： 统计下标0到下标倒数第2，字节组中bit=1的个数，即01000001 01000000  00000000

--》bitcount K1 0 -2　　--》3

 

 注意：redis的setbit设置或清除的是bit位置，而bitcount计算的是byte位置。

 

#### bitop

(1)格式

`bitop and(or/not/xor) <destkey> [key…]`

 ![image-20210521233354520](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233354.png)

bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。

 

(2)实例

2020-11-04 日访问网站的userid=1,2,5,9。

setbit unique:users:20201104 1 1

setbit unique:users:20201104 2 1

setbit unique:users:20201104 5 1

setbit unique:users:20201104 9 1



2020-11-03 日访问网站的userid=0,1,4,9。

setbit unique:users:20201103 0 1

setbit unique:users:20201103 1 1

setbit unique:users:20201103 4 1

setbit unique:users:20201103 9 1

 

计算出两天都访问过网站的用户数量

bitop and unique:users:and:20201104_03

 unique:users:20201103unique:users:20201104

计算出任意一天都访问过网站的用户数量（例如月活跃就是类似这种） ， 可以使用or求并集

 ![image-20210521233419048](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233419.png)

![image-20210521233511077](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233511.png)	

## HyperLogLog

在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站PV（PageView页面访问量）,可以使用Redis的incr、incrby轻松实现。

- 但像UV（UniqueVisitor，独立访客）、独立IP数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为基数问题。

解决基数问题有很多种方案：

（1）数据存储在MySQL表中，使用distinct count计算不重复个数

（2）使用Redis提供的hash、set、bitmaps等数据结构来处理

以上的方案结果精确，但随着数据不断增加，导致占用空间越来越大，对于非常大的数据集是不切实际的。

能否能够降低一定的精度来平衡存储空间？Redis推出了HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

 

- 什么是基数?

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。



### 命令

#### pfadd 

（1）格式

`pfadd <key>< element> [element ...]  添加指定元素到 HyperLogLog 中`

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233818.png" alt="image-20210521233818221" style="zoom:200%;" />

（2）实例

 ![image-20210521233823299](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233823.png)

   将所有元素添加到指定HyperLogLog数据结构中。如果执行命令后HLL估计的近似基数发生变化，则返回1，否则返回0。



#### pfcount

（1）格式

`pfcount<key> [key ...] `计算HLL的近似基数，可以计算多个HLL，比如用HLL存储每天的UV，计算一周的UV可以使用7天的UV合并计算即可

 <img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233829.png" alt="image-20210521233829700" style="zoom:200%;" />

（2）实例

![image-20210521233842928](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233842.png)	

#### pfmerge

（1）格式

`pfmerge<destkey><sourcekey> [sourcekey ...]` 将一个或多个HLL合并后的结果存储在另一个HLL中，比如每月活跃用户可以使用每天的活跃用户来合并计算可得

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233849.png" alt="image-20210521233849288" style="zoom:200%;" />

（2）实例

 ![image-20210521233854478](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521233854.png)



## Geospatial

Redis 3.2 中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。

- redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作

### 命令

#### geoadd

（1）格式

`geoadd<key>< longitude><latitude><member> [longitude latitude member...] ` 添加地理位置（经度，纬度，名称）

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234140.png" alt="image-20210521234140562" style="zoom:200%;" />

（2）实例

geoadd china:city 121.47 31.23 shanghai

geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen 116.38 39.90 beijing

 ![image-20210521234155777](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234155.png)

两极无法直接添加，一般会下载城市数据，直接通过 Java 程序一次性导入。

有效的经度从 -180 度到 180 度。有效的纬度从 -85.05112878 度到 85.05112878 度。

当坐标位置超出指定范围时，该命令将会返回一个错误。

已经添加的数据，是无法再次往里面添加的。

#### geopos 

（1）格式

`geopos <key><member> [member...] `获得指定地区的坐标值

 <img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234214.png" alt="image-20210521234214149" style="zoom:200%;" />

（2）实例

 ![image-20210521234224012](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234224.png)

#### geodist

（1）格式

`geodist<key><member1><member2> [m|km|ft|mi ]` 获取两个位置之间的直线距离

 <img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234242.png" alt="image-20210521234242077" style="zoom:200%;" />

（2）实例

获取两个位置之间的直线距离

 ![image-20210521234251049](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234251.png)

单位：

m 表示单位为米[默认值]。

km 表示单位为千米。

mi 表示单位为英里。

ft 表示单位为英尺。

如果用户没有显式地指定单位参数， 那么 GEODIST 默认使用米作为单位

 

#### georadius

（1）格式

`georadius<key>< longitude><latitude>radius m|km|ft|mi ` 以给定的经纬度为中心，找出某一半径内的元素

 <img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234316.png" alt="image-20210521234316936" style="zoom:200%;" />

经度 纬度 距离 单位

（2）实例

![	](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521234333.png)	



# 配置文件

redis.conf文件

## Units单位

- 配置大小单位,开头定义了一些基本的度量单位，只支持bytes，不支持bit

- 大小写不敏感

![image-20210521231620119](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231620.png)

## **INCLUDES包含**

- 类似jsp中的include，多实例的情况可以把公用的配置文件提取出来

![image-20210521231637857](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231637.png)

## **网络相关配置**

### bind

- 默认情况bind=127.0.0.1只能接受本机的访问请求

- 不写的情况下，无限制接受任何ip地址的访问

````
生产环境肯定要写你应用服务器的地址；服务器是需要远程访问的，所以需要将其注释掉
如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的响应
````

![image-20210521231748987](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231749.png)

### protected-mode

- 将本机访问保护模式设置no

![image-20210521231822945](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231823.png)

### port

- 端口号，默认 6379

![image-20210521231839050](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231839.png)

### tcp-backlog

- 设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。

  在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。

  注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值（128），所以需要确认增大/proc/sys/net/core/somaxconn和/proc/sys/net/ipv4/tcp_max_syn_backlog（128）两个值来达到想要的效果

![image-20210521231920102](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231920.png)

### timeout

- 一个空闲的客户端维持多少秒会关闭，0表示关闭该功能。即永不关闭。

![image-20210521231946329](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521231946.png)

### tcp-keepalive

- 对访问客户端的一种心跳检测，每个n秒检测一次。
- 单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60 

![image-20210521232051883](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232051.png)

## GENERAL通用

### daemonize

- 是否为后台进程，设置为yes; 守护进程，后台启动

![image-20210521232142092](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232142.png)

### pidfile

- 存放pid文件的位置，每个实例会产生一个不同的pid文件

![image-20210521232202958](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232203.png)

### loglevel 

- 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为**notice**

- 四个级别根据使用阶段来选择，生产环境选择notice 或者warning

![image-20210521232237555](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232237.png)

### logfile

- 日志文件名称

![image-20210521232301303](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232301.png)

### **databases** 

- 设定库的数量 默认16，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id

![image-20210521232320215](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232320.png)

## SECURITY安全

### 设置密码

- 访问密码的查看、设置和取消

- 在命令中设置密码，只是临时的。重启redis服务器，密码就还原了。

- 永久设置，需要再配置文件中进行设置。

![image-20210521232349647](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232349.png)



## LIMITS限制

### maxclients

- 设置redis同时可以与多少个客户端进行连接。
- 默认情况下为10000个客户端。
- 如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。

![image-20210521232453937](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232454.png)

### maxmemory

- 建议**必须设置**，否则，将内存占满，造成服务器宕机

- 设置redis可以使用的内存量。**一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。**

- 如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。

- 但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

![image-20210521232552180](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232552.png)

### maxmemory-policy

- volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）
- allkeys-lru：在所有集合key中，使用LRU算法移除key
- volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
- allkeys-random：在所有集合key中，移除随机的key
- volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
- noeviction：不进行移除。针对写操作，只是返回错误信息

![image-20210521232636591](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232636.png)

### maxmemory-samples

- 设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。

- 一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。

![image-20210521232739791](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232739.png)

# 发布与订阅

```
Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis 客户端可以订阅任意数量的频道。
```

1、客户端可以订阅频道如下图

 ![image-20210521232845786](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232845.png)           

2、当给这个频道发布消息后，消息就会发送给订阅的客户端

 ![image-20210521232833847](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232833.png)

 

## 实现

1、 打开一个客户端订阅channel1

`SUBSCRIBE channel1`

 ![image-20210521232948058](C:/Users/jianjian/AppData/Roaming/Typora/typora-user-images/image-20210521232948058.png)             

2、打开另一个客户端，给channel1发布消息hello

`publish channel1 hello`

 ![image-20210521232953481](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232953.png)

返回的1是订阅者数量

3、打开第一个客户端可以看到发送的消息

 ![image-20210521232957495](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210521232957.png)

注：**发布的消息没有持久化，如果在订阅的客户端收不到hello，只能收到订阅后发布的消息**



# Redis事务

- Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

- Redis事务的主要作用就是串联多个命令防止别的命令插队。

## Multi,Exec,discard

从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。

组队的过程中可以通过discard来放弃组队。 

![image-20210525231405024](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231412.png)



![image-20210525231433313](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231433.png)





组队成功，提交成功

![image-20210525231537832](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231537.png)

组队阶段报错，提交失败

![image-20210525231541918](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231541.png)

组队成功，提交有成功有失败情况



## 事务的错误处理

- 组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。

![image-20210525231649509](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231649.png)

- 如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。

![image-20210525231653917](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231654.png)

## 事务冲突

```properties
一个请求想给金额减8000
一个请求想给金额减5000
一个请求想给金额减1000
```

![image-20210525231732874](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231732.png)

### 悲观锁

![image-20210525231802010](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231802.png)

- **悲观锁(Pessimistic Lock)**, 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。**传统的关系型数据库里边就用到了很多这种锁机制**，比如**行锁**，**表锁**等，**读锁**，**写锁**等，都是在做操作之前先上锁。



### 乐观锁

![image-20210525231815785](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525231815.png)

- **乐观锁(Optimistic Lock),** 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis就是利用这种check-and-set机制实现事务的。



## 锁

`WATCH key [key ...] `

- 在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务**执行之前这个(****或这些) key** **被其他命令所改动，那么事务将被打断。**

`unwatch`

- 取消 WATCH 命令对所有 key 的监视。

  如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。

  http://doc.redisfans.com/transaction/exec.html

## 事务三特性

Ø **单独的隔离操作** 

n 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

Ø **没有隔离级别的概念** 

n 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

Ø **不保证原子性** 

n 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 



# 持久化

[官网介绍](http://www.redis.io)

Redis 提供了2个不同形式的持久化方式。

- RDB（Redis DataBase）

- AOF（Append Of File）

## RDB(Redis DataBase)

在指定的时间间隔内将内存中的数据集快照写入磁盘， 也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里

### 备份是如何执行的

Redis会单独**创建（fork）一个子进程来进行持久化**，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。

**RDB**的缺点是最后一次持久化后的数据可能丢失。

### Fork

- Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程

- 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“**写时复制技术**”

- **一般情况父进程和子进程会共用同一段物理内存**，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

### 持久化流程

![image-20210525232752611](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525232752.png)

### dump.rdb文件

- 在redis.conf中配置文件名称，默认为dump.rdb (可以修改)

![image-20210525232828713](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525232828.png)

- rdb文件的保存路径，也可以修改。默认为Redis启动时命令行所在的目录下

  dir "/myredis/"

  ![image-20210525232847008](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525232847.png)

### 配置策略与命令

- 默认策略

![image-20210525232951256](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525232951.png)

3600秒  有1个key变化

30秒 有10个key变化

60秒 有10000个key变化 

满足任何一个条件 便会进行初始化

```properties
save ：save时只管保存，其它不管，全部阻塞。手动保存。不建议。
bgsave：Redis会在后台异步进行快照操作， 快照同时还可以响应客户端请求。
可以通过lastsave 命令获取最后一次成功执行快照的时间
```



`flushall`

执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义

`save`

格式：save 秒钟 写操作次数

RDB是整个内存的压缩过的Snapshot，RDB的数据结构，可以配置复合的快照触发条件，

默认是1分钟内改了1万次，或5分钟内改了10次，或15分钟内改了1次。

- `禁用`

不设置save指令，或者给save传入空字符串

`stop-writes-on-bgsave-error`

当Redis无法写入磁盘的话，直接关掉Redis的写操作。推荐yes.

![image-20210525233421850](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525233421.png)

`rdbcompression压缩文件`

对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。

如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。推荐yes.

![image-20210525233451267](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525233451.png)

`rdbchecksum检查完整性`

在存储快照后，还可以让redis使用CRC64算法来进行数据校验，

但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能

推荐yes.

![image-20210525233523956](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525233524.png)

### 备份

先通过config get dir 查询rdb文件的目录 

将*.rdb的文件拷贝到别的地方

rdb的恢复

- 关闭Redis

- 先把备份的文件拷贝到工作目录下 cp dump2.rdb dump.rdb

- 启动Redis, 备份数据会直接加载

### 优势

- 适合大规模的数据恢复

- 对数据完整性和一致性要求不高更适合使用

- 节省磁盘空间

- 恢复速度快

![image-20210525233700331](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525233700.png)

### 劣势

- Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

- 虽然Redis在fork时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能。

- 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

### 动态停止

RDB：`redis-cli config set save ""#save`后给空值，表示禁用保存策略

### 总结

![image-20210525233814067](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525233814.png)



## AOF(Append Only File)

以**日志**的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(**读操作不记录**)，

**只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

#### AOF持久化流程

  (1）客户端的请求写命令会被append追加到AOF缓冲区内；

（2）AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；

（3）AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；

（4）Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的；

![image-20210525233931170](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525233931.png)

### 开启

`修改默认的appendonly no，改为yes`

- 默认不开启

可以在redis.conf中配置文件名称，默认为 appendonly.aof

AOF文件的保存路径，同RDB的路径一致。

- AOF和RDB同时开启，系统默认取AOF的数据（数据不会存在丢失）

### 启动与恢复

- AOF的备份机制和性能虽然和RDB不同, 但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载。
  - 正常恢复
  - 修改默认的appendonly no，改为yes
  - 将有数据的aof文件复制一份保存到对应目录(查看目录：config get dir)
  - 恢复：重启redis然后重新加载

 

- 异常恢复
  - 修改默认的appendonly no，改为yes
  - 如遇到AOF文件损坏**，通过/usr/local/bin/**redis-check-aof--fix appendonly.aof进行恢复
  - 备份被写坏的AOF文件
  - 恢复：重启redis，然后重新加载

### 同步频率

- `appendfsync always`

始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好

- `appendfsync everysec`

每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。

- `appendfsync no`

redis不主动进行同步，把同步时机交给操作系统。

### Rewrite重写压缩

1是什么：

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof

2重写原理，如何实现重写

AOF文件持续增长而过大时，会`fork出一条新进程来将文件重写(也是先写临时文件最后再rename)`，redis4.0版本后的重写，是指上就是`把rdb 的快照，以二级制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作`。

`no-appendfsync-on-rewrite：`

- 如果 `no-appendfsync-on-rewrite=yes `,不写入aof文件只写入缓存，用户请求不会阻塞，但是在这段时间如果宕机会丢失这段时间的缓存数据。（降低数据安全性，提高性能）

   如果 no-appendfsync-on-rewrite=no, 还是会把数据往磁盘里刷，但是遇到重写操作，可能会发生阻塞。（数据安全，但是性能降低）

- 触发机制，何时重写
  - Redis会记录上次重写时的AOF大小，默认配置是当**AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发**
  - 重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。 

`auto-aof-rewrite-percentage`设置重写的基准值，文件达到100%时开始重写（文件是原来重写后文件的2倍时触发）

`auto-aof-rewrite-min-size：`设置重写的基准值，最小文件64MB。达到这个值开始重写。

- 例如：文件达到70MB开始重写，降到50MB，下次什么时候开始重写？100MB

- 系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,

如果Redis的`AOF当前大小>= base_size +base_size*100% (默认)且当前大小>=64mb(默认)的情况下`，Redis会对AOF进行重写。 

3、重写流程

- bgrewriteaof触发重写，判断是否当前有bgsave或bgrewriteaof在运行，如果有，则等待该命令结束后再继续执行。

- 主进程fork出子进程执行重写操作，保证主进程不会阻塞。

- 子进程遍历redis内存中数据到临时文件，客户端的写请求同时写入aof_buf缓冲区和aof_rewrite_buf重写缓冲区保证原AOF文件完整以及新AOF文件生成期间的新的数据修改动作不会丢失。

- 子进程写完新的AOF文件后，向主进程发信号，父进程更新统计信息。2).主进程把aof_rewrite_buf中的数据写入到新的AOF文件。

- 使用新的AOF文件覆盖旧的AOF文件，完成AOF重写。

![image-20210525235027390](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525235027.png)

### 优势

![image-20210525235045863](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525235045.png)

- 备份机制更稳健，丢失数据概率更低。

- 可读的日志文本，通过操作AOF稳健，可以处理误操作。

### 劣势

- 比起RDB占用更多的磁盘空间。

- 恢复备份速度要慢。

- 每次读写都同步的话，有一定的性能压力。

- 存在个别Bug，造成恢复不能。

### 总结

![image-20210525235132016](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210525235132.png)

## 总结

官方推荐两个都启用。

- 如果对数据不敏感，可以选单独用RDB。

- 不建议单独用 AOF，因为可能会出现Bug。

- 如果只是做纯内存缓存，可以都不用。

### 官网建议

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储

- AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾. 

- Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大

- 只做缓存：如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.

- 同时开启两种持久化方式
  - 在这种情况下,当redis重启的时候会优先载入AOF文件来恢复原始的数据, 因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.

- RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件。那要不要只使用AOF呢？ 
  - 建议不要，因为RDB更适合用于备份数据库(AOF在不断变化不好备份)， 快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段。

- 性能建议
  - 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。
  - 如果使用AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。
  - 代价,一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。
  - 只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。
  - 默认超过原大小100%大小时重写可以改到适当的数值。

# 主从复制



















# 管道

- 使用Jedis包下 Pipeline类实现
- 可以通过封装函数式接口操作此类

### 方式2

```java
```



### 方式1

```java

@Autowired
	RedisTemplate<Object, Object> redis;
	@GetMapping("/redisTest")
	@ResponseBody
	public String test() {
		RedisUtil redisUtil = new RedisUtil(redis);
		int number = 5;
		Long start = System.currentTimeMillis();
		for (int i = 0; i < number; ++i) {
			redisUtil.set(("T" + i), (i + ""));
		}
		Long end = System.currentTimeMillis();
		System.out.println("--" + (end - start));
 
		// 1.executePipelined 重写 入参 RedisCallback 的doInRedis方法
		List<Object> resultList = redis.executePipelined(new RedisCallback<Object>() {
 
			@Override
			public String doInRedis(RedisConnection connection) throws DataAccessException {
				// 2.connection 打开管道
				connection.openPipeline();
				// 3.connection 给本次管道内添加 要一次性执行的多条命令
				for (int i = 0; i < number; ++i) {
					connection.set(("B" + i).getBytes(), (i + "").getBytes());
				}
				// 4.关闭管道 不需要close 否则拿不到返回值
				// connection.closePipeline();
				// 这里一定要返回null，最终pipeline的执行结果，才会返回给最外层
				return null;
			}
		});
		Long start1 = System.currentTimeMillis();
		System.out.println("---" + (start1 - end));
		// 5.最后对redis pipeline管道操作返回结果进行判断和业务补偿
		for (Object object : resultList) {
			System.out.println(object);
		}
		return "test:" + (end - start) + ":" + (start1 - end);
```





