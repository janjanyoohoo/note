# JVM

## Class文件

### 组成

![](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210809233037.png)

> class文件包含三个内容, 内容只有两种数据,结构无符号数或者表.
>
> 表尾_info结尾的结构,无符号数则是u4 u2. 表中包含数据结构(表内部是一个新的数据结构可以有表和无符号数)

### 结构

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

## 虚拟机类加载机制

> 加载->验证->(解析->准备->初始化) 连接->使用->卸载
>
> 解析 准备 初始化 总称为 连接. 一个类的生命周期只有五个部分

![image-20210812223923033](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210812223923.png)

### 加载

- class加载到内存
  - 获取二进制的字节流
  - 静态存储结构(class二进制结构)转换为方法区运行时数据结构
    - tips:会将常量放到方法区的常量池中
  - 在java堆中生成一个class对象,相当于一个句柄可以通过句柄方法区,作为访问方法区的入口

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

- 对类变量分配内存并设置类变量的初始化阶段
  - 仅对static的变量进行内存分配

> static int i = 2
>
> 此时准备阶段赋予值是0,而不是2,因为这个时候<clinit>方法还没有调用,仅赋予默认值
>
> static final i = 2
>
> 此时编译器会将i优化到常量池中,准备阶段便会把值赋予9.因为这个变量属于常量池.

### 解析

- 对符号引用进行解析,把符号引用指向直接引用
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

### 使用

通过对象调用方法...

### 卸载

垃圾回收器释放

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

## 类加载器

> 查找顺序,自下向上, 加载顺序自上向下. 

![image-20210814140755201](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814140755.png)

### 双亲委派模型

![image-20210814151102646](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814151102.png)



### Launcher$AppClassLoader

> 应用加载器,用于加载我们编写的类,从classPath下查找,找不到的类依次向上检查

### Launcher$ExtClassLoader

> 扩展类加载器,AppClassLoader的上层加载器,用于加载类路径java_home/lib/ext下的类

### BootStrapClassLoad

> 根加载器,负责加载java/lib中的类,由C语言实现

## 运行时数据区

- 静态编译: 把Java文件编译成字节码文件Class文件.这个时候Class文件以静态文件的方式存在.
- 类加载器: 把Class字节码文件加载到内存中.

> 类的元数据: 方法描述符 简单名字等存在方法区,存储类的描述信息
>
> 实例信息: 存储在堆中
>
> 

#### 堆

> 唯一的作用用来存放对方实力,GC发生在这个地方.

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

> 栈中存储栈帧,栈帧是程序的调用逻辑

![image-20210814171332959](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814171333.png)

> 虚拟机栈: 分配基本类型和自定义对象的引用
>
> 本地方法栈: 为了Native方法的调用,执行,退出
>
> 部分虚拟机不区分虚拟机栈与本地方法栈以及程序计数器--->合为栈区

![image-20210814172305069](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210814172305.png)

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



## 内存回收

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

