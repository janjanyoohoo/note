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
> 主要涉及: 类,接口,方法.字段等.. 



### 初始化





### JVM内存划分

![image-20210812223620935](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210812223628.png)

- 方法区: 字符串常量池,静态变量
