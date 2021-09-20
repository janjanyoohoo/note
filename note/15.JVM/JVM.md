# JVM

## Class文件

### 哪些类型有Class的对象

- class
- interface
- [] 数组
- enum 
- annotation
- primitive type 基本类型
- void

### 组成

![](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210809233037.png)

> class文件包含三个内容, 内容只有两种数据,结构无符号数或者表.
>
> 表尾_info结尾的结构,无符号数则是u4 u2. 表中包含数据结构(表内部是一个新的数据结构可以有表和无符号数)

### 结构

大致由一下几个部分组成

- 魔数

- Class文件版本
- 常量池
- 访问标识
- 类索引,父类索引,接口索引集合
- 字段表集合
- 方法表集合
- 属性表集合

![image-20210809232311629](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210809232318.png)

- u1 一个字节
- u2 二个字节
- u4 四个字节
- u8 八个字节

> 字节码的具体解释

- u4 代表这是一个class文件,固定ca fe ba be
- u2 副版本号,
- u2 主版本号, 
  - 16进制转2进制51->1.7 / 53->1.8
- u2 常量池计数器
  - 总数-1 为实际常量个数,常量池0位保留未被使用
- cp_info 常量池表
  - u1代表字段的类型
  - u1后面的长度需要根据类型在常量池结构表]进行匹配 不同类型可能u2 u8等

### 常量池

常量池中主要存储字面量与符号引用

> 当虚拟机运行时,需要从常量池中获取对应的符号引用,在类加载过程中的解析阶段将其替换为直接引用,并翻译到具体的内存地址中.

- 字面量

  - 文本字符串
  - 声明为final的常量值

- 符号引用

  > tips:符号引用: 用一组符号来描述所引用的目标,符号引用与虚拟机的内存布局无关,引用的目标不一定加载到了内存中.
  >
  > 直接引用: 可以是直接指向目标的指针,相对偏移量或者是一个能间接定位到目标的句柄.直接引用时和虚拟机实现的内存布局相关的.如果有了直接引用,那么说明引用的目标必定已经存在于内存中了.

  - 类和接口的全限定名
  - 字段的名称和描述符
  - 方法的名称和描述符

> 常量池结构表,二进制标志对应cp_info中对应位置值进制代表的类型

![image-20210809234527346](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210809234527.png)

![image-20210809234633681](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210809234633.png)

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210809234651.webp)

- 字面量
  - 接近java层面常量的定义,如文本字符串,final引用的常量等
- 符号引用
  - 类和接口的全限定名
    - com.jianjian.api...
  - 字段的名称和描述符
    - private public...
  - 方法和名称的描述符
    - private public...
- <init> 与<clinit> 
  - 构造方法,对象实例化时调用
  - 类和接口初始化时调用,所有的类变量和静态初始化语句都会被java编辑收集到<clinit>方法中
    - 如果没有类变量或者静态初始化语句,则不会编译出<clinit>方法
    - 如果仅有类变量并且代码中没有初始化则也不会编译出<clinit>方法

### 虚拟机指令

- invokespecial #1

1. 到常量池中找#1索引常量(#1 MethodRef #4 #22)

> #4 指向常量池4号常量
>
> #22 指向常量池22号常量

2. 找到4号常量(#4 java/lang/Thread)
   1. 此处的Thread为当前class的父类,如果没有父类则此处是Object,此处显示的是父类的类型
3. 找到22号常量(#22)
4.  #22 NameAndType类型 (指向->#7 #8)
5. #7 对应<init>
6. #8 对应 ()V

> 命令执行结果 组合为 V java/lang/Thread.<init>( )  找到构造方法的信息

###  访问标志

![image-20210810232304388](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210810232304.png)

> tips: 访问控制符的值由于ACC_SUPER的缘故,在当前jdk版本中需要在实际的值 | 0x0020对应列表中的值,class文件中的值例如public是0x0001 | 0x0020 = 0x0021

### 类索引/父类索引/接口索引集合

![image-20210810233807053](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210810233807.png)

> tips: 类名(this_class)的长度受到u2长度的限制两个字节FFFF,实际最大是256的长度.

类索引-父类索引-接口计数器(无接口不存在)-接口1(无则不存在)-接口2(无则不存在).....

- `00 03` 代表索引->类索引(全限定)
- `00 04`代表索引-> 父类索引(全限定)
- `00 02`代表接口数量-> 2个接口
- `00 05`常量索引-> 接口1(全限定)
- ...

> 00 03..等为class文件16进制对应结构表位置的数值



### 属性表结构

![image-20210811215159971](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210811215200.png)

- u2 -> 常量池索引
- u4 -> 属性长度
- u1 -> attribute属性个数

> 对照属性表 =(u2 + u4 + u1*length)

### 字段表结构

![image-20210811215729889](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210811215730.png)

1. 方发表查找方法:

- access_flag - 访问修饰符
- name_index - 找到简单名字
- descriptor_index - 找到返回值

2. 其他

- attribute_count - 属性表数量

### Code表



- max_stacks 方法最大栈深
- max_locals 最大方法局部变量数
- code_length 代码行数
- arg_size 普通方法默认有1个参数(this),静态方法无此参数

> 方法内容需要对照字节码指令表

### 异常表

![image-20210811222818230](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210811222818.png)

### 属性表

![image-20210811224551908](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210811224552.png)

### 方法表

![image-20210811225301245](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210811225301.png)

### 本地变量表

- start 开始行号
- length 长度影响行号

> start+length = 本地变量的作用域

- slot 槽

> 变量用几个槽存储

- name 变量名
- signature 签名

> 值是类型,泛型擦除时使用签名,java虚拟机中是伪泛型,在编译时会把泛型擦除可能导致方法不认识. 签名用于标识泛型

> tips : 本地方法有个默认参数this. 所以本地变量表也有个本地变量this

### 字节码

### 字节码指令

> Java虚拟机指令由一个字节长度的,代表着某种特定操作含义的操作码,以及跟随其后的0-多个代表此操作所需要的操作数构成.虚拟机中许多指令不包含操作数,只有一个操作码.

## 类加载

> 加载->验证->(解析->准备->初始化) 连接->使用->卸载
>
> 解析 准备 初始化 总称为 连接. 一个类的生命周期只有五个部分

![image-20210812223923033](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210812223923.png)

### 加载

- - 获取二进制的字节流
  - 静态存储结构(class二进制结构)转换为方法区运行时数据结构
    - tips:会将常量放到方法区的常量池中
  - 在java堆中生成一个class对象,相当于一个句柄可以通过句柄方法区,作为访问方法区的入口

#### Class实例的位置

> Class的构造方法是私有的,只有JVM可以创建

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210905210255.png" alt="image-20210905210255047" style="zoom: 25%;" />

#### 数组类型的加载

> 数组本身并不是由类加载器负责创建,而是由JVM在运行时根据需要而直接创建,但是数组的元素类型依然需要靠类加载器去创建.

- 如果数组的元素是引用类型,那么遵循定义的加载过程递归加载和创建数组A的元素类型
- JVM使用指定的元素类型和数组维度来创建新的数组类
- 如果数组的元素类型是引用类型,数组类的可访问性由元素类型的可访问性决定,否则数组类的可访问性将被缺省定义为`pulbic`

#### 双亲委派



### 验证

- 验证class文件
  - 验证魔数magic_number -> cafebabe
  - 验证jdk版本号
  - 验证常量池(常量类型,常量数据结构)是否正确
  - class文件的每个部分(方法表,字段)是否正确
  - 元数据(父类,继承,final)验证
  - 字节码验证
  - 指令验证(通过指令能否找到方法,字段,类)

### 准备

> 对static 修饰的变量分配内存并赋予默认值.
>
> static final的变量在编译阶段就确定了,准备阶段对已经显示赋值的static final修饰的变量进行赋值.
>
> static修饰的变量则需要等到初始化阶段赋值,此时仅会赋予默认值.涉及到方法调用等操作的static final变量赋值则会等到初始化阶段赋值.

`static int i = 2`

此时准备阶段赋予值是0,而不是2,因为这个时候<clinit>方法还没有调用,仅赋予默认值

`static final i = 2`

此时编译器会将i优化到常量池中,准备阶段便会把值赋予9.因为这个变量属于常量池.

链接阶段对于static final变量赋值时仅会对字面量/常量值的变量赋值,涉及到方法调用等操作的static final变量赋值则会等到初始化阶段赋值.

### 解析

> 多数虚拟机会在初始化步骤完成后执行解析步骤

- 对类,接口,字段,方法 符号引用进行解析,把符号引用指向直接引用
  - 符号引用是以字面量的形式定义在常量池中的

> 直接引用: 指向目标的指针(内存地址/偏移量)
>
> 主要涉及: 类,接口,方法.字段的句柄等.. 

- **字段**的解析, 简单匹配 `简单名字+描述符同时满足`
  - 搜索顺序 : `本类 -> 上层接口 -> 父类 -> Object 依次匹配搜索`
  - 如果找到但没有权限访问(private) 报错 `java.lang.IlleagelAccessError`
  - 未匹配到报错 `java.lang.NoSuchFieldError`

- **类方法**的解析, 简单匹配 `简单名字+描述符同时满足`
  - 搜索顺序 : `本类 -> 父类 -> 上层接口 ->  Object 依次匹配搜索`
  - 在接口中找到方法,但是本类中没有说明当前类是抽象类(1.8此处实现待确认)不能通过对象调用方法,报错`java.lang.AbstractMethodError`
  - 未匹配到报错 `java.lang.NoSuchMethodError`
  - 匹配到但是无权限(private) 报错 `java.lang.IlleagelAccessError`

- **接口方法**的解析
  - 搜索顺序 `本接口->父接口`
  - 未匹配到报错 `java.lang.NoSuchMethodError`
  - 接口没有私有方法,所以没有权限校验

> 当父接口与父类有相同的方法,子类调用该方法时会调用父类的方法, 当子类重写方法时会覆盖父类的方法并同时实现接口的该方法.

### 初始化

> <clinit> 类的初始化,包含静态变量,静态代码块的初始化(ps:如果没有静态变量和静态块则没有<clinit>方法)
>
> <init> 实例的初始化,构造器

类的初始化时类装载的最后一个阶段,如果前面的步骤都没问题,那么表示类可以顺利的装载到系统中. 此时类才会开始执行Java字节码.

初始化阶段的重要工作是执行类的初始化方法<clinit> 方法

- 该方法仅能够由Java编译器生成并由JVM调用,程序开发者也无法自定义一个同名的方法.更无法通过java程序中调用该方法.
- 它由类静态成员的赋值语句以及static语句块合并产生

只有在类中的static变量显示赋值了才会出现此方法.

<init>方法一定会出现在class的method表中

##### 子类加载前先加载父类

> 在加载一个类之前,虚拟机总是会试图加载该类的父类,因此父类的<clinit>方法总是会在子类的<clinit>之前调用,也就是说`父类的static代码块优先级高于子类`

##### 哪些类不会生成Clinit方法

1. shtatic final修饰的变量会使用常量编译的方式不会产生<clinit>方法
   1. static final的类在连接-准备阶段赋值   
2. 没有显示赋值的static变量,没有static静态代码块
3. 成员变量不是static的

##### <clinit>的调用会死锁吗

> 对于<clinit>方法的调用,也就是类的初始化,虚拟机会在内部确保其多线程环境中的安全性

虚拟机会保证一个类的<clinit>方法在多线程环境中被正确的加锁,同步.如果多个线程同时去初始化一个类,那么只有一个线程能够执行<clinit>方法.其他线程需要阻塞等待.直到活动线程执行<clinit>方法完毕.

因为函数<clinit>方法是带锁的线程安全的,所以在一个<clinit>方法中有耗时长的操作,可能造成多个线程阻塞.并且引发死锁.

如果之前的线程成功加载了这个类,则等在队列中的线程就没机会再执行<clinit>方法,当需要使用时,虚拟机会直接返回已经准备好的信息.

##### 类初始化主动使用与被动使用

> 主动使用会触发类的加载执行初始化阶段,loadClass只会加载load阶段不会执行初始化阶段.
>
> 被动的使用,意味着不需要执行初始化阶段,便不会执行<clinit>方法

1. Class.forName("xxx")与getClassLoad().loadClass("xxx")的区别
   1. forName是主动使用, loadClass是被动使用
2. 被动使用
   1. 访问一个静态字段时,只有真正声明这个字段的类才会被初始化.
      1. 通过子类引用父类的静态变量,不会导致子类初始化
   2. 通过数组定义类的引用,不会触发类的初始化
   3. 引用常量不会触发此类或接口的初始化,因为常量在链接阶段就已经被显示赋值了.,
   4. 调用classLoader类的loadClass方法加载一个类,并不是对类的主动使用,不会导致类的初始化.
3. 哪些情况会触发类的加载(主动使用)
   1. 当创建一个类实例,new/反射/clone/序列化
   2. 调用类的静态方法,即使用了invokestatic指令
   3. 使用类/接口的静态字段时候,(final修饰的特殊考虑)
   4. 使用java.lang.reflect包中反射类的方法时,比如Class.forName()
   5. 初始化子类时,如果发现父类还没有初始化则需要先初始化父类.
   6. 如果一个接口定义了default方法,那么直接实现或间接实现该接口的类的初始化,该接口要在其实现之前初始化
   7. 当虚拟机启动时,用户需要指定一个要执行的主类(main()方法类),虚拟机会先初始化这个主类
   8. 初次调用MethodHandle实例时,初始化该MethodHandle指向的方法所在的类.

### 使用 

通过对象调用方法...

### 卸载

> 当类的Class对象不再被引用,即不可触及时,Class对象就会结束生命周期,类在方法区内的数据也会被卸载,从而结束类的生命周期.

方法区主要回收两个内容

- 不再使用的常量
  - 不再被任何地方引用
- 不再使用的类型
  - 该类的所有实例对象都被回收,不再存在任何该类与派生子类的实例
  - 加载该类的加载器已经被回收,此条件苛刻,除非是设计过可替换类加载器的场景,否则一般难以达成.
  - 该类对应的Class对象没有在任何地方被引用,,无法在任何地方通过反射访问该类的方法.

## JVM内存结构

### 运行时内存结构

![image-20210812223620935](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210812223628.png)

> 方法区与堆 是存储数据的地方
>
> 虚拟机栈,本地方法栈,程序计数器 是执行程序逻辑的

- 方法区: 字符串常量池,静态变量

### 内存结构

![image-20210814143944778](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814143945.png)

> 本地方法库 netive方法中封装了一些与操作系统有关的方法.

## 对象

### 对象的实例化

对象的几种创建方法

- new
- Class.newInstance()
  -  jdk9后替换为反射constructor
- 反射constructor的newInstance
- clone()
- 反序列化
- 第三方库Objenesis利用asm字节码技术,动态生成

### 创建对象的步骤

1. 判断对象对应的类是否执行了加载,链接,初始化.

   > 虚拟机遇到一条new命令,首先去检查这个指令的参数能否在Metaspace的常量池中定位到一个类的符号引用,并且检查这个符号引用代表的类是否已经被加载,解析,初始化

   - 如果没有,在双亲委派模型下,使用当前类的加载器以Classload+包名+类名为key进行查找对应的.class文件
   - 如果没有找到文件,则会抛出classNotFoundException异常
   - 找到则进行类加载,并生成对应的Class对象

2. 为对象分配内存

   > 首先计算对象占用的空间大小,接着在堆中划分一块内存给新对象.
   >
   > 如果实例成员变量是引用变量,那么仅分配引用变量空间即可,即4个字节大小.

   - 选择哪种分配方式由Java堆是否规整决定,而Java堆是否规整又与所采用的垃圾回收器是否带有压缩整理功能决定.

   1. 指针碰撞(内存规整)

      > 内存规整时,虚拟机将会使用指针碰撞法来为对象分配内存.意思时所有用过的内存在一边,空闲的内存在另一边,中间放着一个指针作为分界点的指示器,分配内存就仅仅是吧指针指向空闲的那边挪动了一段与对象大小相同的距离.

      如果收集器使用`Serial / ParNew`这种基于`压缩算法`的虚拟机采用指针碰撞法.

      一般使用带有`compact整理过程`的收集器使用指针碰撞;

      <img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210901233221.png" alt="image-20210901233220985" style="zoom:25%;" />

   2. 空闲列表

      > 如果内存不完整,虚拟机需要维护一个列表,使用空闲列表分配.

      内存不完整,已使用的内存与未使用的相互交错,那么虚拟机将采用的是空闲列表方法来为对象分配内存,

      虚拟机维护一个表,记录哪些内存块是可以用的,再分配的使用从列表中找到一块足够大的空间划分给对象实例,并更新空闲列表.这种分配方式称为空闲列表(free list)

      <img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210901233039.png" alt="image-20210901233038780" style="zoom:25%;" /><img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210901233134.png" alt="image-20210901233134297" style="zoom:25%;" />

3. 处理并发安全问题

   > 在分配内存空间时候,另外一个问题是保证new对象的线程安全性, 创建对象是非常频繁的操作,虚拟机需要解决并发安全问题,JVM使用两种方式解决.

   - CAS 失败重试,区域加锁,保证指针更新操作的原子性
   - TLAB 把内存分配的动作按照线程划分在不同的空间中进行,即每个线程在Java堆中预分配一小块内存,称为本地线程分配缓冲区.虚拟机是否使用TLAB,可以通过 `-XX:+/-UseTLAB`参数来设置

4. 初始化分配到的空间

   > 内存分配结束后,虚拟机将分配到的内存空间都初始化为0值,(不包括对象头)这一步保证对象的实例字段在Java代码中可以不用赋值就能够直接使用,程序能够访问到这些字段所对应的零值

   将内存中每个字节的值都设置为0,这个值就是对象的默认值,对应为引用类型的null,基础类型的0等等.

5. 设置对象的对象头

6. 执行<init>方法进行初始化

   >  调用构造方法,仅仅是进行了初始化,对象到此步骤已经在内存中创建完毕

### 对象的内存布局

![image-20210901233948268](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210901233948.png)

- 对象头
  - 运行时元数据
    - 哈希值:对象在堆空间中都有一个首地址值,栈空间的引用根据这个地址指向堆中的对象,这就是哈希值起到的作用.
    - GC分带年龄: 年龄计数器,达到阈值进入OLD
    - 锁状态标志: 偏向锁轻量锁重锁
    - 线程持有的锁
    - 线程偏向ID
    - 偏向时间戳  
  - 类型指针
    - 指向方法区中的类新信息
- 实例数据
- 对齐填充

### 对象的访问定位

- 句柄访问

  - 堆中划分一块内存用作句柄池,reference中存储对象的句柄池中地址.句柄中包含对象实例与类型数据各自具体地址的信息
  - 优点: reference中存储稳定句柄地址,对象被移动(垃圾收集时移动很普遍)时只会改变句柄中实例数据的指针,refernce本身不需要修改

  <img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907221726.png" alt="image-20210907221726531" style="zoom:25%;" />

- 直接指针访问

  - 通过对象的类型指针访问

![image-20210907220521770](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907220522.png)

## 类加载器

#### 作用

ClassLoader是Java核心组件,所有的Class都是通过类加载器加载的.

#### 显示加载与隐式加载

- 显示加载: 代码中通过ClassLoader加载的对象
- 隐式加载: 则是不直接在代码中通过ClassLoader加载Class

#### 唯一性

对于任何一个类,都需要由加载他的类和这个类本身一同确认在Java虚拟机中的唯一性.每一个类加载器,在JVM中都有独立的类名称空间.

比较两个类是否相同,只有在这两个类是同一个加载器加载的情况下才有意义.

- 每个类加载器都有自己的命名空间,命名空间由改类加载器和所有的父加载器所加载的类组成.
- 统一命名空间内,不会出现类的完整名字相同的两个类
- 不同的命名空间中,有可能出现类名称相同的两个类.

> 通过这一个特性可以运行同一个类的不同版本.

#### 基本特征

- 双亲委派模型
- 可见性
  - 子类加载器可以访问父类加载的类型,父类不可以访问子类加载的类型
- 唯一性
  - 父类加载过的类型,子类就不会加载.邻居加载器可以加载 .

> 查找顺序,自下向上, 加载顺序自上向下. 

![image-20210814140755201](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814140755.png)

### 双亲委派模型

- 优点
  - 避免类的重复加载,确保一个类的全局唯一
  - 保护程序安全,防止核心API被随意篡改
- 缺点
  - 检查类是否加载的委托过程是单向的,顶层的ClassLoader无法访问底层ClassLoader所加载的类

![image-20210814151102646](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814151102.png)

### Launcher$AppClassLoader

> 应用加载器,用于加载我们编写的类,从classPath下查找,找不到的类依次向上检查

### Launcher$ExtClassLoader

> 扩展类加载器,AppClassLoader的上层加载器,用于加载类路径java_home/lib/ext下的类

### BootStrapClassLoad

> 根加载器,负责加载java/lib中的类,由C语言实现

### Tomcat类加载器

> Tomcat默认是不支持双亲委派模型的,可通过配置开启
>
> 首先通过Bootstrap类加载器加载,然后跳到WEB-INF/class下加载
>
> tomcat中核心api还是通过bootstrap类加载器遵循双亲委派加载的.

- Common类加载器
  - 加载Tomcat文件夹下的lib文件夹中的类
- Catalina类加载器
  - 加载BootStrap Common等jar中的类
- Shared类加载器
  - 加载webapp公共的类
- WebApp类加载器
  - 隔离加载不同webapp的类
- Jsp类加载器
  - 热加载类加载器

#### Tomcat类加载执行顺序

1. 使用Bootstrap引导类加载器加载
2. 使用System系统类加载器加载
3. 使用应用类加载器加载WEB-INF/classes中类(tomcat中src下的类优先于jar中的类加载)
4. 使用应用类加载器加载WEB-INF/lib中类
5. 使用Common类加载器在CATALINA_HOME/lib中加载

![image-20210905231251445](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210905231251.png)

## 运行时数据区

- 静态编译: 把Java文件编译成字节码文件Class文件.这个时候Class文件以静态文件的方式存在.
- 类加载器: 把Class字节码文件加载到内存中.

> 类的元数据: 方法描述符 简单名字等存在方法区,存储类的描述信息
>
> 实例信息: 存储在堆中
>
> 

#### 堆

一个JVM只存在一个堆,在JVM启动时被创建,其空间大小也就确定了.

<JAVA虚拟机规范>规定堆可以在物理上不是连续的空间,但是在逻辑上是连续的.

在方法结束后,堆中的对象不会马上移除,仅仅在垃圾收集的时候才会被移除. 

所有的数组和堆永远不会存储在栈中,栈中只保存引用,这个引用指向对象或者数组在堆中的位置. 

> 唯一的作用用来存放对象实例,GC发生在这个地方与方法区.

##### TLAB

默认是开启状态的 `-XX:+/-UserTLAB` 默认情况下TLAB的内存空间非常小,占堆的1%.

`-XX:TLABWasteTargetPercent`设置TLAB空间所占Eden空间的百分比大小,一旦对象在TLAB空间分配内存失败,JVM会尝试通过使用加锁机制确保数据操作的原子性从而直接在Eden中直接分配内存.

从内存模型而不是垃圾收集的角度,对Eden区域进行划分,JVM为每个线程分配一个私有的缓存区域.包含在Eden区域中.

堆划分一个线程私有的区域,这部分区域在线程分配对象时避免加锁操作影响性能.

![image-20210906232801507](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210906232801.png)

![image-20210906220412779](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210906220412.png)

![image-20210814165110516](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814165110.png)

#### 方法区

> 存储类的以描述信息,常量池,静态变量和热点代码
> 方法区主要回收两个内容,不再使用的常量与不再被使用的类.

![image-20210907204827368](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907204910.png)

#### 存储内容

> 1. 8将静态变量与字符串常量池存放在堆中,因为字符串常量容易进行回收

- 已被加载的类信息
- 运行时常量
- JIT代码缓存
- 域信息
- 方法信息

![](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907210941.png)

![image-20210907210920028](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907210920.png)

![image-20210814165700051](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814165700.png)

![image-20210814170012064](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814170012.png)

> 动态(即时)编译: 将class字节码文件编译成cpu可以执行的命令.
>
> JIT: 热点代码编译,将编译后的代码存储到方法区.    ps:避免热点执行的代码每次都要编辑.

#### 程序计数器(寄存器)

> 线程独享,记录当前线程执行到的代码位置

寄存器分为两部分:

- 指令寄存器
  - 从程序计数器取下一条指令的地址,cpu每执行一条指令都会从指令寄存器取一条指令执行.
  - 指令寄存器被cpu取走消息后,就会从程序计数器获取下一条指令
- 程序计数器
  - 存放下一条指令的地址

![image-20210814170524648](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814170524.png)

#### 栈

可以通过命令设置栈的最大深度-XX:ThreadStackSize / -Xss size

- 一般默认为512-1024k,取决于操作系统
- 栈的大小直接决定了栈深(函数调用的最大可达深度).

> 虚拟机栈占用的内存设置过大的时,在进程内存不变的情况下会导致进程的线程数变少.

> 栈中存储栈帧,栈帧是程序的调用逻辑. 

![image-20210814171332959](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814171333.png)

> 虚拟机栈: 分配基本类型和自定义对象的引用
>
> 本地方法栈: 为了Native方法的调用,执行,退出
>
> 部分虚拟机不区分虚拟机栈与本地方法栈以及程序计数器--->合为栈区

![image-20210814172305069](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814172305.png)

#### 栈帧的内部结构

##### 操作数栈

> 主要用于保存计算过程的中间结果,同时作为计算过程中临时变量的存储空间

##### 动态链接

![image-20210906210928858](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210906210936.png)

##### 方法的调用 

JVM中,将符号引用转换为调用方法的直接引用时共有两种方式.

- 静态链接
  - 当一个字节码文件被装在进入JVM内部时,如果被调用的目标方法在编译期间可知,且在运行期间保持不变.这种情况下直接将调用方法的符号引用转换为直接引用的过程称之为静态链接.
- 动态链接
  - 如果被调用的方法在编译期间无法确认下,只能够在运行期间将调用方法的符号引用转换为直接饮用.这种引用转换具备动态性,因此称之为动态链接.

方法的绑定机制: 早起绑定 / 晚期绑定.绑定是一个字段,方法,或者类在符号引用被替换为直接引用的过程中.这仅会发生一次.

- 早起绑定
  - 早起绑定指的被调用的目标方法如果在编译期间可知,并且在运行期间保持不变时,即可以将这个方法与所属的类型进行绑定,这样一来由于明确了被调用的目标方法究竟是哪一个,就可以使用静态链接的方式将符号引用转换为直接引用.
- 晚起绑定
  - 如果被调用的方法在编译期间无法确定下来.只能够在程序运行期间根据实际的类型绑定相关的方法.这种方式称之为晚期绑定.(多态等)

#### JVM执行流程

> 调用逻辑: 
>
> 加载类放入方法区,创建对象并将方法区类信息句柄存放在堆的对象中,
>
> 持有堆中对象引用,通过对象引用调用方法,根据对象引用找到堆中对象,
>
> 根据对象中存放的方法区信息引用定位到方法区类信息,获取方法的字节码执行.

![image-20210813225223091](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210813225223.png)

![image-20210814224441865](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814224442.png)

![](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814172811.png)

### 空间分配担保

在发生MinorGC前,虚拟机会检查老年代最大可用连续空间是否大于新生代所有对象的总空间.

- 如果大于,则MinorGC是安全的
- 如果小于,虚拟机检查-XX:HandlePromotionFailure设置的值是否允许担保失败
  - 如果=true,那么会检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小,如果大于则会尝试一次MonirGC.但是这次GC是由风险的,如果小于或者=false则会进行一次FullGC
  - 1.6以后默认为true

## 内存回收

![image-20210907231235516](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907231235.png)

### 什么是垃圾

> 指在运行程序中没有任何指针指向的对象,这个对象就是需要被回收的垃圾

 

### FullGC MinorGC  MojorGC

- 一种部分收集
- 一种整堆收集

- 部分收集: 不是完整收集整个Java堆的垃圾收集;又分为
  - 新生代收集:(MinorGC / YoungGC) 只是新生代的回收
  - 老年代收集:(MajorGC / OldGC) 只是老年代的收集
    - 目前只有CMS会有单独收集老年代的行为
    - 注意: 多数时MajorGC会和FullGC混淆使用,需要具体分辨是老年代回收还是整堆的回收.
  - 混合收集:(MixedGC) 收集整合新生代和部分老年代的垃圾收集
    - 目前只有G1会有这种行为
- 整堆收集(FullGC)
  - 收集真个Java堆和方法区的垃圾收集

> 垃圾回收发生在堆与方法区内. 栈区属于线程私有数据所以不会进行回收.

>  打印GC日志信息命令
>
> > -XX:PrintGCDetails  -XX:UseSerialGC  //打印GC信息,序列化执行
> >
> > -XX:PrintGC //简略的GC信息
> >
> > -XX:P日内同
> >
> > GC日志解读:
> >
> > ![image-20210814232609799](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814232609.png)

### GC触发时机

#### 年轻代MinorGC

- 当年轻代空间不足,就会触发MinorGC.这里的年轻代满指的是Eden去满.Survivor区不会导致GC
- 因为Java对象大多数具备朝生夕灭的特性,所以MinorGC非常频繁,且速度快.
- MinorGC会引发STW,暂停其他用户的线程,等待垃圾回收结束,用户线程才会回收完毕.

![image-20210906230655233](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210906230655.png)

#### 老年代MajorGC

- 发生在老年代的GC对象从老年代消失时MajorGC或者FullGC发生了.
  - 出现MajorGC经常伴随一次MinorGC(并非绝对),
  - 在老年代空间不足时,会先尝试触发MinorGC如果之后空间还不足,则触发MajorGC
- MajorGC的速度一般会比MinorGC慢10倍以上,STW的时间更长.
- 如果MajorGC后,内存还不足就会抛出OOM;

#### 整堆回收FullGC

- 触发FullGC的情况有5种
  - 调用System.GC()是系统建议执行GC,但是不一定会执行
  - 老年代空间不足
  - 方法区空间不足
  - 通过MinotGC后进入老年代的平均大小大于老年代的可用内存
  - 由Eden,from向to区复制,对象大小大于to可用内存,则把该对象转到老年代,且老年代内存小于该对象大小.

> 开发中应该尽量避免FullGC

### OOM解决

1. 解决OOM或者Heap space的异常,首先通过内存影响分析工具(Eclipse Memory Analyzer)对dump出来的堆转储快照进行分析,重点是确认内存中对象是否必要,也就是要先分清楚是`内存泄漏`还是`内存溢出`.
2. 如果是内存泄漏,可进一步通过工具查看泄漏对象的GC ROOTS引用连,能够找到泄漏对象是通过怎么样的路径与GC ROOTS相关联导致垃圾收集器无法自动回收他们,掌握了泄漏对象的类型信息,以及GC ROOTS引用连信息可以定位泄漏代码位置.
3. 如果是内存溢出,则应当减产-Xms -Xmx与物理机器内存是否可以调大.从代码上检查是否有些对象生命周期过长,持有状态时间过长情况,尝试减少程序运出行期间的内存消耗.

### 内存区域

![image-20210815224529076](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815224529.png)

### Java引用类型

- 强引用
  - A  a = new A() 常规引用属于强引用,只要引用还在就不会回收对象
- 软引用
  - 如果内存够用则不回收,内存不够发生内存溢出前,进行回收. 如果这次回收还没有足够的内存则抛出OOM
  - 这一类对象是Java.lang.ref包下的提供的类, new出的对象是软件用
- 弱引用
  - 生存到下一次垃圾回收前,无论当前内存是否够用,都会回收到弱关联的引用
  - java.lang.ref提供了相关类的实现
- 虚引用
  - 不会对对象的声明周期有影响.也无法通过它得到对象实例,唯一作用是在对象被垃圾回收前收到一个系统通知



> 如果判断一个对象是否还能被使用到? 引用计数与可达性分析

### 垃圾判别阶段算法

#### 引用计数

java未使用此算法

给对象添加一个引用计数器,每当对这个对象进行一次引用,计数器就会+1,每当引用失效的时候,引用计数器-1,每当这个计数器=0的时候,表示这个对象不会再被引用.

> 循环引用->  A内部引用B  B内部引用A A置位null,B置位null.

- 优点
  - O(N)
- 缺点
  - 无法处理循环引用.导致内存泄漏

![image-20210907232048996](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907232049.png)

#### 可达性分析

- 基本思路
  - 内存中存活对象都会被根对象集合直接或间接连接着,搜索所走过的路径称为引用链.
  - 如果目标对象没有任何引用相连接,则是不可达的,就意味着该对象已经死亡,可以标记为垃圾对象
  - 可达性算法中,只有能够被根对象集合直接或间接连接的对象才是存活对象

![image-20210907233651916](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210907233652.png)

##### GC Roots包含的元素

- 虚拟机栈中引用的对象
  - 各个线程被调用的方法使用到的参数,局部变量等
- 本地方法栈内JNI(Ntive方法)引用的对象
- 类静态属性引用的对象
  - Java类的引用类型静态变量
- 方法区中常量引用的对象
  - 比如字符串常量池,String table中的引用
- 所有被同步锁Synchronized持有的对象
- Java虚拟机的内部引用
  - 基本数据类型对应的Class对象,一些常驻的异常对象,系统类加载器
- 反应Java虚拟机内部情况的JMXBean,JVMTI中注册的回调,本地代码缓存等

> Root采用栈的方式存放变量和指针,所以如果一个指针,他保存了堆内存中的对象,但是自己又不存放在堆内存中,那么他就是一个Root.

> 如果使用可达性分析算法来判断内存是否可回收,那么分析工作必须在一个能够保障一致性的快照中进行.这点不满足的花分析结果准确性就无法保证.
>
> 可达性算法在GC时必须Stop then world ,即是在CMS中枚举根节点也是必须要停顿的

##### 四种对象

- 虚拟机栈中,栈帧中的本地变量表中引用的对象

> void func(){
>
> A a = new A();   //本地变量表中引用的对象
>
> }

- 方法区中类静态属性引用的对象
- 方法区中常量引用对象
- 本地方法栈中JNI(即一般的Native方法)引用的对象

>  private native void sort(Person person)
>
> private void test(){
>
> ​	Person person = new Person();
>
> ​	sort(person); //native方法引用的对象 
>
> }

##### GC类型

在new对象时候,是否要记录对象在内存中的地址等信息.

- 保守型GC
  - 在new对象时不进行任何记录,只有在GC扫描时开始递归扫描对象引用. 效率低
- 半保守型GC
  - 在类中记录相关信息,在扫描时只要有足够的信息就可以进行回收
- 精确型GC
  - 主流虚拟机使用
  - OopMap,存放运行时的信息,类型信息地址信息等. 在回收时查询OopMap中存储的信息来判断可以回收的对象

![](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815205749.png)

### 垃圾清除阶段算法

#### 标记清除

> 分为两个阶段,标记阶段,清除阶段. 将可达对象标记出来,在标记完成后统一清除**未被标记**的对象

- 缺陷: 
  - 效率问题: 标记和清除的效率都低.
  - 空间问题: 最主要的会产生内存的空间碎片.

> 当有一个大对象需要一块连续的内存空间,但是空间碎片导致没有连续的空间存放对象便会抛出OOM

![image-20210908221320131](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210908221320.png)

![image-20210815223320551](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815223320.png)

#### 复制算法

> 将内存分为大小完整的两个块,标记A块中的对象清理完成后,将存活对象拷贝到B块.

缺陷:

- 内存缩小为原来一半,利用率低

![image-20210908221303432](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210908221303.png)

![image-20210815223751468](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815223751.png)

#### 标记整理(压缩)算法

> 标记阶段相同,增加一个整理算法

- 优点
  - 消除了标记-清除算繁重内存碎片的缺点.
  - 消除了复制算法中,内存减半的高额代价
- 缺点
  - 效率上低于复制算法
    - 效率低,要标记所有的存货对象,还要整理所有存活对象的引用地址
    - 老年代每次都有大量的存货对象,极为负重
  - 移动过程中,如果对象被其他对象引用,需要调整引用的地址
  - 移动过程中,需要暂停用户线程STW

![image-20210908221135817](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210908221136.png)

![image-20210815223851445](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815223851.png)

#### 分代收集算法

- Mark阶段开销与存活对象的数量成正比
- Sweep阶段的开销与所管理区域的大小成正相关
- Compact阶段的开销与存活对象的数据成正比

> CMS使用Mark-Sweep,回收效率高低延迟,但是碎片化严重 ,当内存回收不佳,碎片导致Concurrent Mode Failure时,将会采用Serial Old执行Full GC 达到对老年代的整理.

- 老年代基于标记清理或者标记整理算法来回收,效率较低
  - 老年代的空间较大,一般都是GB级别,  
- 新生代一半对象的生命周期较短回收频繁,使用复制算法就可以效率的完成
  - 默认大小64M, 
  - MinorGC 小内存GC
- 默认经过15轮gc存活的对象,会进入老年代 (从to->from->to在幸存代中每次GC会来回转移存活代数+1 )

![image-20210815224552876](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815224553.png)

#### 增量收集算法

垃圾收集线程只收集一小片区内存空间,接着切换到应用程序线程.依次反复,直到垃圾收集完成.

### 安全点与安全区域

程序执行时并非在所有地方都能够停顿下来开始GC,只有在特定的位置才能停顿下来开始GC,这些称之为安全点.

>  安全点是进行GC的点,OopMap

1. 抢先式中断
   1. 缺点 sleep wait的线程不会到达安全点
   2. 目前没有虚拟机采用
2. 主动式中断
   1. 设置一个安全点,线程执行到安全点时会停止

![image-20210816221618115](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816221737.png)

#### 安全区域

Safepoint保证了程序执行时,不太长的时间就会遇到可以进入GC的Safepoint

但是线程处于sleep或者clocked状态,这时候线程无法响应JVM的中断请求,也不会走到安全点挂起,JVM也不会唤醒等待线程,这种情况需要安全区域解决.

> 安全区域指的是在一段代码片段中,对象的引用关系不会发生变化,在这个区域的任何位置开始GC都是安全的,那么Safe Region 看做扩展了的Safepoint

实际执行:

- 线程运行到Safe region代码时,首先表示已经进入安全区域,如果这段时间发生GC,JVM会忽略标识位Safe region的线程.
- 线程即将离开Safe region时,JVM会检查是否已经完成GC,完成了则继续运行, 未完成则必须等待收到可以安全离开Safe Region的信号为止.

### 内存泄漏与内存溢出

> OOM之前一般都会有FullGC,但是如果大数组JVM判断后也会不GC直接OOM

#### 内存泄漏的八种情况

- 静态集合类
  - 静态生命周期与类相同,一个不再使用的静态集合,不会被回收
- 单例模式
- 内部类持有外部类
  - 内部类未销毁,外部类便不会销毁
- 各种连接,如数据库连接,网络连接,IO连接
- 变量不合理作用域
- 改变哈希值
- 缓存泄漏
- 监听器和回调

### STW

> Stop The World 指的是GC事件发生过程中,会产生应用程序的停顿.停顿产生时整个应用程序都会被暂停,没有任何响应.类似于卡死的感觉,这个停顿称之为STW.

- 可达性分析算法中枚举根节点(GC Roots)会导致Java执行线程停顿
  - 分析工作必须在一个能确保一致性的快照中进行
  - 一致性指整个分析期间,整合执行系统看起来像是冻结在某个时间点
  - 如果分许过程中对象引用关系还在不断变化,则分析结果的准确性无法保证.
- 被STW中断的应用线程会在完成GC后恢复,频繁中断会让用户感觉像是网速不快卡
- STW时间和采用的垃圾收集器无关,所有的都有STW
- G1同样有STW,但是尽可能的缩短时间
- STW是JVM在后台自动发起和完成的,在用户不可见的情况下,把用户正常工作的线程暂停

### 并发,并行

- 并发

> 吞吐量大

![image-20210908230443539](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210908230443.png)

- 并行

  > 用户线程与垃圾线程同时执行,不一定是并行的可能是交替执行,垃圾回收线程不会停顿用户程序的运行

![image-20210908230257884](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210908230258.png)

## 垃圾收集器

![image-20210816232630876](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210901225023.png)

### 参数

```c
-XX:+Use{垃圾回收器名称}GC  //指定使用的垃圾回收器
-XX:+PrintGCDetails   //打印GC日志
-XX:+PrintGC			//打印简单GC日志
```

> 新生代和老年代都会STW

### Serial 串行

> 单线程的收集
>
> 新生代中使用的都是复制算法,内存小速度快
>
> Old 使用标记整理(压缩)算法

![image-20210909234427418](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210909234427.png)

![image-20210816224247267](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816224247.png)

### ParNew并行

> 并行收集,但是老年代仍然是单线程收集的,收集过程中STW
>
> -XX:ParallelGCThreads 限制线程数量,默认开启和CPU核数相同的线程数

![image-20210909235034596](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210909235035.png)

![image-20210816225810546](C:/Users/jianjian/AppData/Roaming/Typora/typora-user-images/image-20210816225811778.png)

### Parallel Scavenge吞吐优先

- 自适应调节策略, 是Parallel Scavenge和ParNew的重要区别

- 主要优点: 可以控制代码的吞吐量

吞吐量计算公式: 运行用户代码的时间 / (运行用户代码的时间+垃圾收集时间) 可以控制GC时用户线程停顿的时间

```java
-XX:UseParalleGC  //指定年轻代使用并行收集
-XX:UseParalleOldGC //指定老年代使用并行回收期
-XX:ParalleGCThreads   //设置年轻代的并行收集器线程数,默认情况下CPU核数小于8 线程=核心数,
    					//核数大于8,线程= 3 + ((5 * CPU) / 8)
-XX:MaxxGCPauseMillis    //最大垃圾收集停顿(STW)时间(大于0毫秒数),一般不建议设置
-XX:GCTimeRatio			//吞吐量大小,(大于0且小于100整数,吞吐量百分比),默认99, 取值范围0-100, 
    					//计算规则  1 / (N + 1)
-XX:+UseAdaptiveSizePolicy	//内存调优委托给虚拟机管理
```

例: 吞吐量99% = (用户代码运行99分钟 / (用户代码运行99分钟 + GC时间) = 100分总)

> 并行的垃圾收集器,

### Parallel Old

> 对老年代并行收集的收集器,采用标记-压缩算法

![image-20210816232212035](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816232212.png)

### Serial Old收集器

![image-20210816232126657](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816232127.png)

### Concurrent Mark Sweep垃圾收集器

Hotspot第一款真正意义上的并发收集器

搭配Serial或者ParNew

> CMS使用标记-清除算法,这会导致产生一些内存碎片,从而不能使用指针碰撞进行内存分配,只能后使用空闲列表

- 优点
  - 并发收集
  - 低延迟
- 缺点
  - 会产生内存碎片
  - 无法处理浮动垃圾
  - 对CPU资源敏感

**适用场景:Web**

> 只能够在老年代适用
>
> -XX:+UserConsMarkSweep   
>
> -XX:CmsInitiatingOccupanyFraction  //设置堆内存使用的阈值.达到后开始进行回收(1.6以上默认92%)
>
> //内存增长缓慢时可以设置一个稍大的值,大的阈值可以降低CMS触发的频率.如果应用程序内使用频率增长很快应该,则应该降低这个阈值,以避免触发老年代Serial Old串行化收集. 通过这个选项可以有效降低FULL GC 的次数
>
> -XX:+UseCMSCompactFullCCollection   //指定在执行完FullGC后对内存空间进行压缩整理,避免内存碎片(增加停顿时间)
>
> -XX:CMSFullGCsBeforeCompaction   设置在执行多少次Full GC后对内存进行压缩整理

- 开启CMS收集器 `-XX:+UseConcMarkSweepGC`

![image-20210824225148520](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210824225156.png)

- 初始标记(STW)
  - 仅标记GC Roots的直接关联对象
  - CMS在初始标记时标记根节点为`黑色`,子节点标记为`灰色`.
- 并发标记
  - 从GC Roots的子节点开始遍历
  - 在并发标记时,CMS继续标记,将子节点标记为黑色,子子节点标记为灰色,递归标记.未被标记到的则是`白色,`说明不可达.
- 重新标记(STW)
  - 进行STW,停止用户线程,防止在重新标记时用户线程产生新的垃圾
  - 进行可达性分析,并发标记环节中可能`部分对象可能不可达变为不可达`
  - 对于被标记为清理对象后又被进行赋值引用的对象重新标记;例如`a = null; ...  a=new A();`
  - 此事可达对象变为不可达称为浮动垃圾,浮动垃圾需要在下一次GC时才会清理
- 并发清除

##### 优点

> 低停顿

- -XX:UseCMSCompactAtFullCollection 在FullGC后进行一次整理
- -XX:CMSFullGCsBeforeCompaction 在执行多少次FullGC之后进行一次整理操作
- -XX: ...其他参数

##### 缺点

> 并发清理容易产生碎片,可以通过参数修改

### G1收集器

为了适应不断扩大的内存和不断增加的处理器数量.

> 在延迟可控的情况下获得尽可能高的吞吐量 

G1仍然属于分代收集器,只是把内存空间分为多个Region,最大优点是空间整合.

> G1的新生代老年代S1S0等分布与传统垃圾收集器不同

- 开启G1 -XX:UseG1GC

G1提供三种垃圾回收模式,Young GC ,Mixed GC Full GC

#### GC

- 年轻代GC(Young GC)
- 老年代并发标记过程 Concurrent Marking
  - **初始标记阶段**
    - 标记从根节点可达的对象,这个阶段STW并且会触发一次Young GC
  - **根区域扫描Root Region Scanning**
    - G1 GC扫描Survivor 区直接可达的老年代区域对象, 并标记为被引用对象,这一过程必须在Young GC之前完成
  - **并发标记Concurrent Marking** 
    - 在整个堆中进行并发标记,此过程可能被Young GC中断,此阶段若发现区域对象中所有对象都是垃圾.那这个区域会被立即回收.同时并发标记过程还会记录这个区域对象存活的比例
  - **再次标记Remark** 
    - 由于应用程序持续进行,需要修正上一次标记的结果,是STW的,G1采用了比CMS更快的初始快照算法(SATB)
  - **独占清理Cleanup STW,**
    - 计算各个区域存货对象和GC回收的比例并进行排序识别可以混合回收的区域为下个阶段准备
    - 这个阶段并不会去做垃圾收集
  - **并发清理** 
    - 识别并清理完全空闲的区域
- 混合回收(Mixed GC)
  - 整个新生代的回收,还会回收一部分的老年代的回收(不是全部老年代).
  - Mixed GC 并不是Full GC
  - 不能正常进行Mixed GC就会触发FullGC

> 如果需要,单线程独占式高强度的FullGC还是继续存在的,他针对GC的评估失败提供一种保护机制,即强力回收

![image-20210911163842188](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210911163842.png)

#### 分区Region

- -XX:G1HeapRegionSize   
  - //设置每个Region的大小,值是2的幂,范围是1-32MB之间,目标是根据最小的Java堆大小划分出约2048个区域,默认是`堆内存的1/2000`
  - 在JVM生命周期内Region大小不会再次变化
- 一个Region可能属于Eden,Survivor 或者Old 但是一个Region只能属于一个角色.
- G1增加了一种新的内存区域,Humongous 内存区域,主要存储大对象, 如果超过了1.5个Region就会存储到H
  - 为了解决一个短期存在的大对象可能会被直接放进老年代,G1划分Homongous区域,专门存放大对象,如果H区装不下大对象,那么G1会寻找连续的H区来存储,**为了能找到连续的H区域,有时候不得不启动Full GC**,G1大多数行为H区域作为老年代看待.

#### 空间整合

- CMS 标记-清除算法,在若干次GC后进行一次碎片整理
- G1 将内存划分为一个个Region,内存的回收时以Region为单位的,Region之间是复制算法,整体上看实际是标记-压缩算法.两种算法都可以避免碎片,这种特性有助于程序长时间运行,分配大对象时不会因为无法找到连续的内存空间而提前触发下一次GC.尤其当Java堆非常大的时候,G1的优势更加明显.

#### 可预测的停顿事件模型

这是G1另一大又是,G1能建立可预测的停顿时间模型,让使用者明确指定在一个M毫秒的时间段片段内,消耗在垃圾回收商的时间不得超过N毫秒.

- 分区的原因,G1可以只选取部分区域进行内存回收,这样缩小了回收的范围,因此对于全局停顿情况的发生能得到较好的控制.

#### 使用

1. 开启G1
2. 设置堆最大内存
3. 设置最大停顿时间

#### 优化

- 年轻代大小
  - 避免使用-Xmn 或者 -XX:NewRation 等参数设置年轻代大小
  - 固定年轻代大小会覆盖暂停时间目标
- 暂停时间不要过于严苛
  - G1 GC的吞吐量目标是90%应用程序时间,10%垃圾回收时间
  - 评估G1 GC吞吐量时,暂停时间目标不要太严苛,目标严苛表名愿意承受更多的垃圾回收开销,这些会影响到吞吐量

#### 命令

> -XX:+UserG1
>
> -XX:G1HeapRegionSize   //设置每个Region的大小,值是2的幂,范围是1-32MB之间,目标是根据最小的Java堆大小划分出约2048个区域,默认是`堆内存的1/2000`
>
> -XX:MaxGCPauseMillis  设置期望达到的最大GC停顿时间指标.(JVM会尽力实现而不保证一定),默认是200ms
>
> -XX:ParalleGCThread  设置STW时GC的线程数的值,最多为8
>
> -XX:ConcGCThreads 设置并发标记线程数.将N设置为垃圾回收线程数(XX:ParalleGCThread  )的1/4左右
>
> -XX:InitiatingHeapOccupancyPercent  设置触发并发GC周期的Java堆占用率阈值,超过此值,就会触发GC. 默认45

![](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210824234007.png)

![image-20210825204635742](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210825204635.png)

 #### YoungGC

1. 扫描根 GC Roots
2. 更新RememberSet 记录回收对象的数据结构
3. 检测RemeberSet哪些数据需要从年轻代到老年代,或者需要到幸存代
4. 拷贝对象,to幸存代,to老年代

### 垃圾收集器组合关系与使用区域

> Serial - ParNew - Parallel Scavenge 只能在新生代适用
>
> CMS - SerialOld - ParallelOld 只能在老年代使用
>
> ParNew 可以和三种老年代收集器组合使用.

![image-20210816232630876](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816232631.png)



## JVM内存分配

> 对象会首先进入新生代Eden区,大对象则直接进入老年代

![image-20210825215408152](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210825215408.png)

- 大对象进入老年代

  > -XX:PretenureSizeThreshold=n 如果对象尺寸大于这个阈值(字节),则直接进入老年代
  >
  > 注意:这个参数只能在ParNew和Serial这两款收集器起作用

- 长期存活对象

  > -XX:MaxTenuringThreshold 一个对象经理多少次GC会进入老年代,默认值15

- 对象年龄动态判断

  > 如果在Survivor空间中相同年龄所有对象大小综合大于Survivor的一半,年龄大于等于该年龄的对象可以直接进入老年代.无语等到MaxTenuringThreadshold中要求的年龄.

- 空间分配担保

  > HandlePromotionFailure 检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小,如果大于,将尝试进行一次MinorGC 如果小于或者设置不允许冒险.那么将执行一次FullGC

  1. 在MinorGC之前,检查老年代最大可用连续空间是否大于新生代所有对象的大小
  2. 执行MinorGC
  3. 如果空间不够
     1. 检查HandlePromotionFailure是否开启
        1. 未开启->执行FullGC ->仍旧不够 OOM
        2. 开启-> 检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小.
        3.  如果大于,则试着执行MinorGC
        4. 如果小于,则FullGC

  **最终目的减少FullGC次数**

  ![image-20210825214241478](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210825214241.png)

> 如果经过FullGC还是没有足够的空间则OOM

## Java字节码执行

### 字节码执行模式

> java字节码是混合执行模式(解释执行和编译执行)

![image-20210826223417329](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210826223417.png)

### 运行时栈结构

![image-20210826223353890](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210826223354.png)

### 局部变量表

![image-20210826225722446](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210826225722.png)

#### 本地变量执行逻辑

> 本地变量表中标识本地变量存储在那个slot中,32位以内的变量占用1个槽.64的占用2个.
>
> 执行方法到变量时,虚拟机会从本地变量表load变量,压入到操作数栈中执行.

操作数栈与虚拟机栈都是栈结构,本地变量表不是.

![image-20210826231918333](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210826231918.png)

> long 与 double 在32位系统中需要2个slot存储,不是原子的操作. 64位操作系统不存在这个问题.

### 操作数栈

![image-20210826230921746](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210826230922.png)

### 方法返回地址

- 方法遇到返回指令 return
- 方法遇到异常

![image-20210826231103809](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210826231104.png)

## 命令行工具

`jps` 查看当前进程信息

`jstack pid`  查看pid下的线程信息



## 调优

### 调优的基本问题

1. 为什么要调优

- 防止出现OOM,进行JVM规划和预调优
- 解决程序运行中各种OOM
- 减少fullGC出现的频率.解决运行慢,卡顿问题

2. 调优的大方向

- 合理编写代码
- 充分并合理的使用硬件资源
- 合理的进行JVM调优

3. 不同阶段的考虑

- 上线前
- 运行阶段
- 线上出现OOM

4. 总结

- 调优基于实际的业务场景
- 基于监控的调优

### 命令

- -Xms -Xmn堆内存的最小值与最大值
- -XX:NewRatio=2 表示新生代占1,老年代占2. 新生代占堆1/3(**默认**)
- -XX:ServivorRatio 设置Survicor区与Eden比例.
- -Xmn200m 设置新生代内存大小,直接调整内存大小
- -XX:SurvivorRation=8 调整Survivor区空间比例. **默认**是8:1:1
- -XX:HandlePromotionFailure=true 空间分配担保策略. 1.6后默认为true
- -XX:MaxTenuringThreshold=n 默认=15,对象晋升到老年代的阈值
- -XX:PrintGCDetails
- -XX:PrintFlagsFinal
- -XX:MetaspaceSize=Nm  N是元空间的大小
- -XX:+HeapDumpOnOutOfMemoryError   -XX:HeapDumpPath=路径/文件名.hprof /生成dump文件
- -XX:TraceClassLoading    打印加载类的累心
- -XX:TraceClassUnloading  卸载类信息打印
- -XX:+/-DoEscapeAnalysis  启用/禁用逃逸分析栈上分配调优,默认启用
- -XX:+/-PrintEscapeAnalysis  打印逃逸分析的筛选结果
- -XX:+/-UseAdaptiveSizePolicy  自适应调节Eden From to区的大小,ParallelGC 默认开启.此参数不要和XX:ServivorRatio 共同使用会导致参数失效

### GC 日志分析

#### 命令

- verbose:gc 输出GC日志信息,默认输出到标准输出
- -XX:+PrintGC 输出GC日志,类似于verbose:gc
- -XX:+PrintGCDetails 在发生垃圾回收时,打印内存回收详细的日志.并在进程退出时输出当前内存各区域的分配情况
- -XX:+PringGCTimestamps 输出GC发生时的时间戳 
- -XX:+PrintGCDateStamps 输出GC发生时的时间戳,以日期时间的格式
- -XX:+PrintHeapAtGC 在每次GC前后,打印堆信息
- -Xloggc:<file>  表示吧GC日志写到一个文件中去,而不是标准打印输出

### OOM

#### 堆溢出

- 原因
  - 代码中可能UC你在大对象的分配
  - 可能存在内存泄漏,导致多次GC以后无法找到一块足够大的内存容纳当前对象
- 解决方法
  - 检查是否有大对象的分配,最有可能的是大数组的分配
  - 通过jmap命令,把堆内存dump下来,使用MAT工具分析,检查是否存在内存泄漏问题,通过jvisualvm等工具分析dump文件
    - GC log文件分析
    - dump文件分析
  - 如果没有找到明显的内存泄漏,使用-Xmx 加大堆内存
  - 检查是否有大量的自定义Finalizable对象,也可能是框架内部存在的,考虑其存在的必要性

#### 元空间溢出

> 元空间通常进行常量池中常量回收与类型卸载,类型卸载很少发生

- 原因
  - 运行时加载大量代理类,导致方法区被撑爆,无法卸载
  - 应用长时间运行没有重启
  - 元空间内存设置过小
- 解决方法
  - 检查是否元空间设置过小
  - 检查代码中是否有大量的反射操作
  - dump之后通过mat检查是否存在大量反射生成的代理类

#### Overhead limit Exceeded

- 原因
  - JDK6新增加的错误类型,一般是堆太小导致的, SUN官方对此定义超过98%的时间用来GC,并且回收了2%不到的堆内存时会抛出这个异常,本质上是一个预判性的异常,抛出该异常时内存并没有真正的溢出
- 解决方法
  - 检查项目中是否有大量的死循环或有使用大内存的代码.优化代码
  - 添加参数-XX:-UserGCOverheadLimit禁用,但是这个并非解决问题而是将OOM延迟抛出
  - dump内存检查是否存在内存泄漏,如果没有则加大内存.

#### 线程溢出

> 可开启线程数 = (MaxProcessMemory - JVMMemory - ReservedOsMemory)  / ThreadStackSize
>
> MaxProcessMemory : 最大寻址空间(32位系统为4GB,64位系统最大值目前物理机器达不到.) 建议使用64位系统
>
> JVMMemory : VM内存
>
> ReservedOsMemory: 保留的操作系统内存
>
> ThreadStackSize:线程栈的大小
>
> 32位系统中严格遵守此公式,64位系统受到系统设置影响,并非严格按照此公式

64位操作系统受到操作系统pid最大值,最大线程数限制 

### 调整堆大小提高吞吐量

1. 修改Tomcat配置

tomcat不建议直接在catalina.sh里面配置变量,而是卸载catalina同级目录(bin目录)下的setnev.sh中

![image-20210912153605274](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210912153605.png)

> 通过jstat 查看进程gc信心, 如果频繁发生ygc与fgc,可以尝试提高堆空间大小或者S0S1比例进行优化,减少GC次数

### JIT优化之逃逸分析

堆是分配对象的唯一选择

#### 栈上分配

> -XX:+/-DoEscapeAnalysis  启用/禁用逃逸分析栈上分配调优,默认启用
>
> -XX:+/-PrintEscapeAnalysis  打印逃逸分析的筛选结果
>
> 栈上分配启用后,会判断方法中的变量时候发生了逃逸,只要发生了逃逸就会使用栈上分配,与方法一起出栈时被清理

没有发生逃逸的对象(对象仅在方法内部创建和使用,不是传参,不会返回,不会赋值给外部变量)将进行栈上分配

发生逃逸的对象将会分配在堆上

> 开发中能使用局部变量的,就不要再方法外部定义
>
> 栈上分配的对象使用字节方式呈现,会分配出现大量的数组

#### 同步省略(消除) 

方法内部使用局部变量加锁,**在编译成class文件中不会消除,在JIT编译的代码中会消除锁.**

![image-20210912163134593](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210912163135.png)

#### 标量替换

`-XX:+/-EliminateAllocations` 是否开启标量替换, 前提是`-XX:+/-DoEscapeAnalysis` 必须开启

- 开启逃逸分析后开启标量替换才会生效
- 逃逸分析使用标量替换来实现优化,环比标量替换的花逃逸分析优化效果基本不明显

- 标量替换的数据存放在栈上

### 合理配置堆内存

增加内存提升性能是显著的,但是内存设置过大的时候FULL GC的时间就会变长, 内存设置小时就会频繁Full GC 吞吐量下降.

年轻代的内存过大同样面临这样的问题

适当修改S0S1区域的比例也可以避免对象过早的大量进入Old Gen

#### 官方推荐

- Java整个堆: 大小设置,Xmx 和Xms设置为老年代存活对象的3-4倍,即FullGC之后老年代占用内存的3-4倍.
- 方法区: (永久代/元空间)设置为老年代存货对象的1.2 - 1.25倍
- 年轻代: -Xmn 设置为老年代存活对象的1-1.5倍
- 老年嗲: 设置为老年代存货对象的2-3倍.

> 这并不是一个绝对的参考值,实际情况中应当根据dump文件进行分析.查看GC的频率,GC停顿的耗时,内存实际数据,FullGC基本上是不能够有的.然后去做一个合理的分配

如何计算老年代存活的对象大小

JVM参数中添加输出GC日志文件,GC日志中会记录每次FullGC之后各代的内存大小,观察老年代GC之后的空间大小(观察一段时间1-2天)的FullGC之后内存情况,根据多次的FullGC之后的老年代空间大小数据来预估FullGC之后老年嗲存活对象大小.(可以取平均值)

#### 如何计算YGC的频率

例如从数据库读取一条数据占用128个字节,需要读取1000条数据那么一次读取到内存得大小就是`(128B / 1024Kb /1024Mb) * 1000 = 0.122M` 

那么程序并发读取每秒读取100次,内存占用0.122 * 100 = 12.2M,

如果堆内存设置1G,那么年轻代大小333M,那么333M * 80% / 12.2M = 21.8s .

也就是说程序每分钟大约发生2-3此YGC.可以进行一个大致的估算

### 新生代与老年代的比例

-XX:+/-UseAdaptiveSizePolicy 动态的Eden From To区大小,不能和ServivorRatio共同使用.会导致参数失效.

JDK1.8中,使用CMS的话,无论如何设置,UseAdaptiveSizePolicy 都是false.Parallel是默认开启的.

由于UseAdaptiveSizePolicy 会动态调整大小,有些情况Survivor会被调整的很小,可能会导致部分对象直接进入老年代,从而触发FallGC频率增加.

> 面对外部大流量,低延迟系统不建议开启此参数.

> 开启此参数后Eden From To区比例将不会是固定的8:1:1 而是动态调整的
>
> 在使用ParallelGC情况下,此参数不管是否开启,默认Eden From To区的比例都是6:1:1

### CPU占用高

1. ps aux | grep java 查找到当前java进程使用CPU,内存,磁盘的情况,获取使用量异常的进程id
2. top -Hp 进程pid  检查当前使用异常线程pid
3. 方式1: jstack 进程pid > xxx.log 输出log文件, 把线程pid变为16进制 搜索log文件中线程操作信息
4. 方式2 : jstack 进程pid | grep -A20 16进制pid 得到进程相关的信息
   1. -A20:  A=after的意思,找到的位置往后20行, 20可以根据需求修改

> 核心通过jstack 命令行工具来进行分析

- 死锁
  - 定时锁定时释放
- 死循环等

### G1并发线程数对性能的影响

 `-XX:ConcGCThreads=1`主要会对吞吐量有一定的影响.

关注点是GC次数,GC时间,以及Jmeter的平均响应时间

### 调整垃圾回收器提高服务的吞吐量

单核系统使用串行Serial收集器

根据不同的硬件使用不同的回收器

### 日均百万的订单JVM参数设置

#### 服务器配置

计算每个请求产生的数据量 计算大约YGC产生的频率进行机器扩容等优化

#### 控制响应时间在一个范围内



### 面试小结

1. CPU经常100%

一定有线程占用系统资源,还有可能是垃圾回收线程占用100%

2. 系统内存飚高

一方面: jmap -heap , jstat gc 等日志信息

另一方面: dump文件分析

3. 如何监控JVM

命令行工具

图形化界面

4. 

## 性能指标与监控

### GC评估指标 

> 现在JVM调优标准: 在满足最大吞吐量的情况下,降低停顿时间.

- 吞吐量=运行用户代码时间/ (运行用户代码时间+运行垃圾收集时间)

- 垃圾收集开销: 吞吐量的补数,垃圾收集器所占用时间与总时间的比例
- 暂停时间: 执行垃圾收集时,程序工作线程被暂停的时间
- 收集频率: 相对于应用程序的执行,手机操作发生的频率
- 内存占用: Java堆区所占用内存大小
- 快速: 一个独享从诞生到被回收经历的时间

吞吐量与响应时间,内存占用三者是无法同时满足的.

![image-20210908234639072](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210908234639.png)

![image-20210908234817325](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210908234817.png)

高吞吐量程序让应用程序的最终用户感觉程序线程再做生产性的工作,直觉上吞吐量越高的程序运行越快.

低延迟(低停顿时间)较好因为从最总用户角度来看不管是GC还是其他原因导致应用被挂起体验是不好的.

吞吐量和低暂停时间是一对相互竞争的目标

- 选择吞吐量优先,则必然需要降低内存回收的执行频率,从而导致GC需要更长的暂停时间进行回收
- 低延迟优先那么为了降低内存回收的暂停时间,只能频繁的进行回收,但这导致年轻代内存的缩减和UN图来弄个下降.

在设计GC算法时,一个GC算法只可能针对两个目标之一,或尝试折中方案.

### 调优的基本问题





### 调优监控的依据





### 性能优化的步骤





### 性能评价/测试指标

