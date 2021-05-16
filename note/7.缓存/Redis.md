

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









# 管道

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

