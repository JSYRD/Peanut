# Java Review

## 目录：

* ### [Java 语言基础](#1)

  * #### [Java简介](#1.1)

  * #### [Java类型](#1.2)

  * #### [Java引用](#1.3)

  * #### [Java控制](#1.4)

* ### [Java 面向对象](#2)

* ### [Java 进阶内容](#3)

---

<h2 id='1'>Java语言基础</h2>

* <h3 id='1.1'>Java 简介</h3>

  * #### Java的特点：

    * 面向对象(OOP)

      > 支持面向对象编程语法
      >
      > 是使用广泛的面向对象语言之一

    * 跨平台(WORA)

      > 使用Java虚拟机(Java Virtual Machine, or JVM) 和 Java bytecode 实现

    * 类C语法

      > 去除了指针
      >
      > 自动内存管理和垃圾回收(GC)

  * #### Java编写：

    * 流程

      > 编写源代码
      >
      > 编译：javac
      >
      > 运行：java

    * 可能出现的错误

      > 编译时错误 Compile-time error
      >
      > 运行时错误 Run-time error
      >
      > 逻辑错误 Logic error

* <h3 id='1.2'>Java类型</h3>

  * #### 基本类型

    | Primitive type | Size (bits) |  Minimum  |      Maximum       | Wrapper type |    Default     |
    | :------------: | :---------: | :-------: | :----------------: | :----------: | :------------: |
    |    boolean     |      -      |     -     |         -          |   Boolean    |     false      |
    |      char      |     16      | Unicode 0 | Unicode $2^{16}-1$ |  Character   | '\u0000'(null) |
    |      byte      |      8      |  $-128$   |       $+127$       |     Byte     |    (byte)0     |
    |     short      |     16      | $-2^{15}$ |    $+2^{15}-1$     |    Short     |    (short)0    |
    |      int       |     32      | $-2^{31}$ |    $+2^{31}-1$     |   Integer    |       0        |
    |      long      |     64      | $-2^{63}$ |    $+2^{63}-1$     |     Long     |       0L       |
    |     float      |     32      | IEEE 754  |      IEEE 754      |    Float     |      0.0f      |
    |     double     |     64      | IEEE 754  |      IEEE 754      |    Double    |      0.0d      |
    |      void      |      -      |     -     |         -          |     Void     |       -        |

    > 注：
    >
    > 1. boolean只有`true`和`false`两种取值，而对于boolean类型的大小，并没有给出精确的定义，对于不同的虚拟机可能会出现8 bits或者32 bits
    >
    > 2. Java种float和double均按照IEEE 754标准存储，其上下限与IEEE 754标准完全一致，即：
    >
    >    |  类型  |  有效数字  |       最小正正规数        |         最大正数         |
    >    | :----: | :--------: | :-----------------------: | :----------------------: |
    >    | float  | **约**7位  |  约$1.175\times10^{-38}$  | 约$3.402\times10^{+38}$  |
    >    | double | **约**16位 | 约$2.225\times 10^{-308}$ | 约$1.797\times10^{+308}$ |
    >
    >    需要注意的是，有效数字并不是单指小数点后，而是整个数的有效数字。浮点数的精度是有限的。[IEEE754标准: 三, 为什么说32位浮点数的精度是"7位有效数" - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/343040291)
    >
    > 3. **Java使用Unicode字符集的UTF-16格式，而不是ASCII字符集。**所以Java的char并不是8 bits，而是16 bits
    >
    > 4. 由于Java使用JVM运行，所以（除boolean以外），在不同的操作系统中，基本类型不会有区别。（事实上boolean除了可能size有变化以外，也不会有任何区别）
    >
    > 5. Java没有unsigned
    >
    > 6. 注意不可以用数字直接代替boolean,例如
    >
    >    ~~`if(1),if(0)`~~
    >
    >    是错误的，应该为
    >
    >    `if(true),if(false)`

  * #### 数组

    * ##### 初始化：

      > * 静态初始化
      >
      > ```java
      > int []a = {1,2,3,4,5};
      > ```
      >
      > * 动态初始化
      >
      > ```java
      > int []a = new int[5];
      > MyType []m = new MyType[3];
      > 
      > int []a = new int[] {1,2,3,4,5};
      > MyType []m = new MyType[] {
      >     new MyType(),
      >     new MyType(),
      >     new MyType()
      > };
      > ```
      >
      > * 多维数组
      >
      > ```java
      > int [][]a = new int[2][3];
      > ```
  
    * ##### 数组的特性：

      > **数组是对象**，是一种特殊的对象。
      >
      > 可以使用`arrayName.length`来获得数组的长度，可见`length`是它的一个数据成员。
  
  * #### 类
  
    * ##### 定义：
  
      ```java
      class Mytype{
          int i;
          double d;
          char c;					//Fields
          
          void setD(double x);
          double getD();			//Methods
      }
      ```
  
    * ##### 构造对象，访问对象：
  
      ```java
      public class Main{
          public static void main(String []args){
              MyType a = new MyType();
              int b = a.i;
              a.setD(10.0);
              double c = a.getD();
          }
      }
      ```
  
* <h3 id='1.3'>Java 引用</h3>



<h2 id='2'>Java面向对象</h2>

* ### 面向对象编程概述

  * #### 从实际问题到计算模型：

    * 对于给定的某实际问题（如计算两个向量的内积）

      > 面向过程编程需要定义函数，实现函数并控制计算流程。问题是每个数组参数都需要一个长度参数，向量的实现方式也不确定。抽象程度不高。
      >
      > 面向对象编程是从对象本身出发，首先将问题转化为不同的对象，再考虑对对象本身进行变换。并且，不同的对象可以有不同的功能，对象之间也可以传递消息。**抽象程度更高。**

  * #### 面对对象语言：

    * 编程语言直接提供了对对象的支持

      > 定义、构造对象，对对象的操作，对象提供的服务以及消息传递方法等

    * 提高了问题的抽象程度，缩短了实际问题到计算机算法的距离。*（图源ybwu.org，如侵删）*

    ![image-20220602162122013](D:/MyOwnData/Hope/GitHub/ECNUCS_Programming-Club/Clubbbbbb/Review/JavaReview.assets/image-20220602162122013.png)

  * #### 面向对象编程要素：

    > * 任何事物都是对象
    > * 程序为一些对象间的相互协作
    > * 一个对象可以包含另一个对象
    > * 每个对象都有类型
    > * 同一类型的对象接收相同类型的消息，提供相同类型的服务

  * #### 对象：

    * **对象的基本要素：状态、行为和类型**
    
    * 对象的状态(State)
    
      > * 每一个对象都有自己的状态
      > * 程序可以改变一组对象的状态
    
    * **对象的接口(Interface)**
    
      > * 对象向外界提供的服务，“行为”
      > * 接口的实现
      > * **隐藏实现的细节**（封装，Encapsulation，梦中情码就是所有接口完美的封装）
      > * *In any relationship, it's important to have boundaries that are respected by all parties involved.* **不该看的不看**
    
    * 对象的类型 (Type, class)
    
      > 同类型的对象就是一组行为相同但可能状态不同的对象。
      >
      > 可类比基本类型，但是更加多元化。
      >
      > 类型或类是图纸，对象是按照图纸制造的机器。
    
  * #### 类与类之间：
  
    * Has-a
    * Is-a
    * 多态



<h2 id='3'>Java进阶内容</h2>




<h2 id='1'>Java语言基础</h2>

