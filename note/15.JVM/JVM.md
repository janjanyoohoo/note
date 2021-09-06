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
- 实例数据
- 对齐填充

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

### 引用计数

给对象添加一个引用计数器,每当对这个对象进行一次引用,计数器就会+1,每当引用失效的时候,引用计数器-1,每当这个计数器=0的时候,表示这个对象不会再被引用.

> 循环引用->  A内部引用B  B内部引用A A置位null,B置位null.

### 可达性分析

- 可达性分析算法

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

![image-20210815205749079](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815205749.png)

### 标记清除

> 分为两个阶段,标记阶段,清除阶段. 在标记完成后统一清除被标记的对象

- 缺陷: 
  - 效率问题: 标记和清除的效率都低.
  - 空间问题: 最主要的会产生内存的空间碎片.

> 当有一个大对象需要一块连续的内存空间,但是空间碎片导致没有连续的空间存放对象便会抛出OOM

![image-20210815223320551](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815223320.png)

### 复制算法

> 将内存分为大小完整的两个块,标记A块中的对象清理完成后,将存活对象拷贝到B块.

缺陷:

- 内存缩小为原来一半,利用率低

![image-20210815223751468](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815223751.png)

### 标记整理算法

> 标记阶段相同,增加一个整理算法

![image-20210815223851445](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815223851.png)

### 分代收集算法

> 常用的算法,

- 老年代基于标记清理或者标记整理算法来回收,效率较低
  - 老年代的空间较大,一般都是GB级别,  
  - FullGC
- 新生代一半对象的生命周期较短回收频繁,使用复制算法就可以效率的完成
  - 默认大小64M, 
  - MinorGC 小内存GC
  - MajorGC 大内存GC
- 默认经过15轮gc存活的对象,会进入老年代 (从to->from->to在幸存代中每次GC会来回转移存活代数+1 )

![image-20210815224552876](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210815224553.png)

### Savepoint安全点

>  STW :stop the world(正常执行的用户线程全部停止)
>
>  安全点是进行GC的点,OopMap

1. 抢先式中断
   1. 缺点 sleep wait的线程不会到达安全点
2. 主动式中断

![image-20210816221618115](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816221737.png)

## 垃圾收集器

![image-20210816232630876](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210901225023.png)

### 参数

```c
-XX:+Use{垃圾回收器名称}GC  //指定使用的垃圾回收器
-XX:+PrintGCDetails   //打印GC日志
-XX:+PrintGC			//打印简单GC日志
```

> 新生代和老年代都会STW

### Serial 串行收集器

> 单线程的收集
>
> Ed +S1+S0 使用的都是赋值算法,内存小速度快
>
> Old 使用标记整理算法

![image-20210816224247267](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816224247.png)

### ParNew收集器

> 并行收集,但是老年代仍然是单线程收集的,收集过程中STW

![image-20210816225810546](C:/Users/jianjian/AppData/Roaming/Typora/typora-user-images/image-20210816225811778.png)

### ParallelScavenge收集器

- 主要优点: 可以控制代码的吞吐量

吞吐量计算公式: 运行用户代码的时间 / (运行用户代码的时间+垃圾收集时间) 可以控制GC时用户线程停顿的时间

```java
-XX:MaxxGCPauseMillis    //最大垃圾收集停顿时间(大于0毫秒数)
-XX:GCTimeRatio			//吞吐量大小,(大于0且小于100整数,吞吐量百分比)
-XX:+UseAdaptiveSizePolicy	//内存调优委托给虚拟机管理
```

例: 吞吐量99% = (用户代码运行99分钟 / (用户代码运行99分钟 + GC时间) = 100分总)

> 并行的垃圾收集器,

### ParallelOld收集器

> 对老年代并行收集的收集器

![image-20210816232212035](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816232212.png)

### Serial Old收集器

![image-20210816232126657](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210816232127.png)

### Concurrent Mark Sweep垃圾收集器

**适用场景:Web**

> 只能够在老年代适用

- 开启CMS收集器 `-XX:+UseConcMarkSweepGC`

![image-20210824225148520](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210824225156.png)

- 初始标记(STW)
  - CMS在初始标记时标记根节点为`黑色`,子节点标记为`灰色`.
- 并发标记
  - 在并发标记时,CMS继续标记,将子节点标记为黑色,子子节点标记为灰色,递归标记.未被标记到的则是`白色,`说明不可达.
- 重新标记(STW)
  - 进行STW,停止用户线程,防止在重新标记时用户线程产生新的垃圾
  - 进行可达性分析
  - 对于被标记为清理对象后又被进行赋值引用的对象重新标记;例如`a = null; ...  a=new A();`
- 并发清除

##### 优点

> 低停顿

- -XX:UseCMSCompactAtFullCollection 在FullGC后进行一次整理
- -XX:CMSFullGCsBeforeCompaction 在执行多少次FullGC之后进行一次整理操作
- -XX: ...其他参数

##### 缺点

> 并发清理容易产生碎片,可以通过参数修改

### G1收集器

G1仍然属于分代收集器,只是把内存空间分为多个Region,最大优点是空间整合.

> G1的新生代老年代G1G0等分布与传统垃圾收集器不同

- 开启G1 -XX:UseG1GC

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

## 命令

`jps` 查看当前进程信息

`jstack pid`  查看pid下的线程信息



## 调优

### 命令

- -Xms -Xmn堆内存的最小值与最大值

- -XX:NewRatio=2 表示新生代占1,老年代占2. 新生代占堆1/3(**默认**)
- -Xmn200m 设置新生代内存大小,直接调整内存大小
- -XX:SurvivorRation=8 调整Survivor区空间比例. **默认**是8:1:1
- -XX:HandlePromotionFailure=true 空间分配担保策略. 1.6后默认为true
- -XX:MaxTenuringThreshold=n 默认=15,对象晋升到老年代的阈值
- -XX:PrintGCDetails
- -XX:PrintFlagsFinal

### 生产中的问题





### 调优的基本问题





### 调优监控的依据





### 性能优化的步骤





### 性能评价/测试指标

