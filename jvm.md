

#  进程

## 基本概念

操作系统资源分配的基本单位

## 进程包含的维度

a.独立运行的程序

b.独立的数据空间

## 进程的实现方式

操作系统内核维护一个进程表，也称为进程控制块（PCB）。进程表项为一个进程启动的上下文信息，包括进程管理（寄存器，PC，PSW，调度信息，打开文件的状态等）、存储管理（代码段、数据段、堆栈段指针等）、文件管理（目录、PID等）信息，进程切换运行态时会把上下文信息读取或写入 PCB

## 进程的状态

运行态: 正在CPU上运行的状态

就绪态：已就绪但没有被调度程序选中

阻塞态：进程在等待资源的状态

创建态：操作系统为进程分配资源,初始化进程表（PCB）

终止态：操作系统回收进程所拥有的资源,撤销PCB

# 线程

## 基本概念

线程是任务执行的最小单元,所有线程共享进程的资源

## 线程的实现方式

用户级线程（用户管理）

内核级线程（操作系统内核管理）

# CPU调度算法

## 非抢占式的先来先服务算法（FCFS）

按照进程就绪的先后顺序使用CPU

特点：公平，实现简单，但是长进程后面的短进程需要等待很长时间，不利于用户体验。

## 非抢占式的最短作业优先（SJF）

具有最短完成时间的进程优先执行

## 最短剩余时间优先（SRTN）

SJF抢占式版本，即当一个新就绪的进程比当前运行进程具有更短完成时间时，系统抢占当前进程，选择新就绪的进程执行。

特点：改善短作业的周转时间，但如果源源不断有短任务到来，可能使长的任务长时间得不到运行，产生饥饿现象。

## 最高相应比优先算法（HRRN）

是一个综合算法，调度时，首先计算每个进程的响应比R，之后总是选择R最高的进程执行

响应比R=（等待时间+处理时间）/处理时间

## 时间片轮转调度算法

每个进程被分配一个时间片，允许该进程在该时间段运行，如果在时间片结束时该进程还在运行，则剥夺CPU并分配给另一个进程，如果该进程在时间片结束前阻塞或结束，则CPU立即进行切换。

## 虚拟轮转法

主要基于时间片轮转法进行改进，解决在CPU调度中对于I/O密集型进程的不友好。其设置了一个辅助队列，对于I/O型进程执行完一个时间片之后，则进入辅助队列，CPU调度时总是先检查辅助队列是否为空，如果不为空总是优先调度辅助队列里的进程，直到为空，才调度就绪队列的进程

# JVM内存结构

![java内存结构](https://raw.githubusercontent.com/fontStep/photos/master/20200703101640.png)

## 程序计数器

### 基本概念

程序计数器是一块较小的内存空间，它的作用可以看作是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

备注：如果正在执行java方法，程序计数器记录的当前虚拟机字节码指令的地址，如果正在执行native方法，则为空,同时不会产生OOM异常

### 理解

```javaj a
public class Solution {

    public static void main(String[] ags) {

        int a = 100;
        int b = 100;

        System.out.println(a+b);
    }
}
```

通过 Javap -c Solution 查看汇编文件

```java
Compiled from "Solution.java"
public class com.wjw.limiting.mains.Solution {
public com.wjw.limiting.mains.Solution();
        
    Code:
        0: aload_0
        1: invokespecial #1                  // Method java/lang/Object."<init>":()V
        4: return

public static void main(java.lang.String[]);
        Code:
        0: bipush        100
        2: istore_1
        3: bipush        100
        5: istore_2
        6: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        9: iload_1
        10: iload_2
        11: iadd
        12: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        15: return
        }
```

code列是字节码指令的**偏移地址**,可以简单理解为程序计数器所对应的字节码指令地址

关于字节码指令代表的含义理解链接

https://segmentfault.com/a/1190000008722128

https://www.jianshu.com/p/d95cfde7fc49

## Java虚拟机栈

### 基本概念

Java虚拟机栈是线程私有的,**生命周期和线程一致**，描述Java方法执行的内存模型,每个方法的执行过程对应的栈桢的入栈和出栈,栈桢包括局部变量表,操作数栈,动态链接,方法出口等结构

### 栈桢

![栈桢结构](https://raw.githubusercontent.com/fontStep/photos/master/20200703101710.png)

#### 局部变量表

##### 基本概念

局部变量表是一组变量值存储空间，用户存放方法参数和方法内部定义的局部变量,**并且在Java文件编译为class文件时,就已经确定了该方法所需要分配的局部变量表的最大容量**

局部变量不会被系统赋予初始值,类变量在**类加载的准备阶段**会被赋予初始值

##### 变量槽

局部变量表的容量是以变量槽为最小单位,**每个变量槽都可以存储32位长度的内存空间,例如boolean,byte,char,short ,int,float,reference,对于64位长度的数据类型（long,double），虚拟机会以高位对其方法为其分配两个连续的Slot空间，也就是相当于读写会被分割为两次32位读写**

##### slot复用

slot的变量超出了作用域,那么下一次分配slot的时候,将会覆盖原来的数据,slot对对象的引用会影响GC(slot被复用的话,将不会被回收)

##### 理解

例子一：b元素未超过作用域的情况下

```java
package com.wjw.limiting.mains;

/**
 * slot复用测试
 *
 * @author wangjiawei
 */
public class Slot {


    public static void main(String[] args) {

        /**
         * 20 * 1024 * 1024字节 = 20m
         */


        byte[] b = new byte[20 * 1024 * 1024];


        int a = 10;

        System.gc();
    }
}

```

使用javap指令查看汇编文件 

javap -v Slot.class

```java
Classfile /Users/wangjiawei/Documents/project/limiting/target/classes/com/wjw/limiting/mains/Slot.class
  Last modified 2020-5-15; size 533 bytes
  MD5 checksum 586a6eb8aa2e99ae497206625f94216f
  Compiled from "Slot.java"
public class com.wjw.limiting.mains.Slot
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #5.#24         // java/lang/Object."<init>":()V
   #2 = Integer            20971520
   #3 = Methodref          #25.#26        // java/lang/System.gc:()V
   #4 = Class              #27            // com/wjw/limiting/mains/Slot
   #5 = Class              #28            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               LocalVariableTable
  #11 = Utf8               this
  #12 = Utf8               Lcom/wjw/limiting/mains/Slot;
  #13 = Utf8               main
  #14 = Utf8               ([Ljava/lang/String;)V
  #15 = Utf8               args
  #16 = Utf8               [Ljava/lang/String;
  #17 = Utf8               b
  #18 = Utf8               [B
  #19 = Utf8               a
  #20 = Utf8               I
  #21 = Utf8               MethodParameters
  #22 = Utf8               SourceFile
  #23 = Utf8               Slot.java
  #24 = NameAndType        #6:#7          // "<init>":()V
  #25 = Class              #29            // java/lang/System
  #26 = NameAndType        #30:#7         // gc:()V
  #27 = Utf8               com/wjw/limiting/mains/Slot
  #28 = Utf8               java/lang/Object
  #29 = Utf8               java/lang/System
  #30 = Utf8               gc
{
  public com.wjw.limiting.mains.Slot();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/wjw/limiting/mains/Slot;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=3, args_size=1  //注释 locals=3代表有3个局部变量
         0: ldc           #2                  // int 20971520
         2: newarray       byte
         4: astore_1       //注释 将第二步newarray创建的变量放在slot1中
         5: bipush        10
         7: istore_2       //注释 将10放入slot2中
         8: invokestatic  #3                  // Method java/lang/System.gc:()V
        11: return
      LineNumberTable:
        line 18: 0
        line 21: 5
        line 23: 8
        line 24: 11
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      12     0  args   [Ljava/lang/String;
            5       7     1     b   [B
            8       4     2     a   I
    MethodParameters:
      Name                           Flags
      args
}
SourceFile: "Slot.java"

```



例子二：b元素超过作用域的情况下 slot会被复用

```java
package com.wjw.limiting.mains;

/**
 * slot复用测试
 *
 * @author wangjiawei
 */
public class Slot {


    public static void main(String[] args) {

        /**
         * 20 * 1024 * 1024字节 = 20m
         */


        {
            byte[] b = new byte[20 * 1024 * 1024];
        }


        int a = 10;

        System.gc();
    }
}

```

```java
警告: 二进制文件Slot包含com.wjw.limiting.mains.Slot
        Classfile /Users/wangjiawei/Documents/project/limiting/target/classes/com/wjw/limiting/mains/Slot.class
Last modified 2020-5-15; size 514 bytes
        MD5 checksum 636b736ff68dd0d65dfb9f9727e304a8
        Compiled from "Slot.java"
public class com.wjw.limiting.mains.Slot
        minor version: 0
        major version: 52
        flags: ACC_PUBLIC, ACC_SUPER
        Constant pool:
        #1 = Methodref          #5.#22         // java/lang/Object."<init>":()V
        #2 = Integer            20971520
        #3 = Methodref          #23.#24        // java/lang/System.gc:()V
        #4 = Class              #25            // com/wjw/limiting/mains/Slot
        #5 = Class              #26            // java/lang/Object
        #6 = Utf8               <init>
   #7 = Utf8               ()V
           #8 = Utf8               Code
           #9 = Utf8               LineNumberTable
           #10 = Utf8               LocalVariableTable
           #11 = Utf8               this
           #12 = Utf8               Lcom/wjw/limiting/mains/Slot;
           #13 = Utf8               main
           #14 = Utf8               ([Ljava/lang/String;)V
           #15 = Utf8               args
           #16 = Utf8               [Ljava/lang/String;
           #17 = Utf8               a
           #18 = Utf8               I
           #19 = Utf8               MethodParameters
           #20 = Utf8               SourceFile
           #21 = Utf8               Slot.java
           #22 = NameAndType        #6:#7          // "<init>":()V
           #23 = Class              #27            // java/lang/System
           #24 = NameAndType        #28:#7         // gc:()V
           #25 = Utf8               com/wjw/limiting/mains/Slot
           #26 = Utf8               java/lang/Object
           #27 = Utf8               java/lang/System
           #28 = Utf8               gc
           {
public com.wjw.limiting.mains.Slot();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
        stack=1, locals=1, args_size=1
        0: aload_0
        1: invokespecial #1                  // Method java/lang/Object."<init>":()V
        4: return
        LineNumberTable:
        line 8: 0
        LocalVariableTable:
        Start  Length  Slot  Name   Signature
        0       5     0  this   Lcom/wjw/limiting/mains/Slot;

public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
        stack=1, locals=2, args_size=1     //注释 本地变量表中有两个元素
        0: ldc           #2                  // int 20971520
        2: newarray       byte
        4: astore_1          //将第二部newarray创建的变量放入 slot1中
        5: bipush        10
        7: istore_1       //将操作数栈中的10放入slot1
        8: invokestatic  #3                  // Method java/lang/System.gc:()V
        11: return
        LineNumberTable:
        line 19: 0
        line 23: 5
        line 25: 8
        line 26: 11
        LocalVariableTable:
        Start  Length  Slot  Name   Signature
        0      12     0  args   [Ljava/lang/String;
        8       4     1     a   I
        MethodParameters:
        Name                           Flags
        args
        }
        SourceFile: "Slot.java"
        
```

#### 操作数栈

操作数栈的每一个元素可以是任意的Java数据类型,包括long和double,32的数据类型所占栈容量为1,64位数据类型占用的栈容量为2

当方法开始执行的时候,这个方法的操作数栈时空的,在方法执行的过程中,会有各种字节码指令往操作数栈中入栈/出栈操作

在感念模型里,栈桢之间时应该是相互独立的,不过大多数虚拟机都会做一些优化处理，使局部变量表和操作数栈之间有部分重叠,这样在进行方法调用的时候可以直接共用参数，而不需要做额外的参数复制等工作

![局部变量表和操作数栈重叠](https://raw.githubusercontent.com/fontStep/photos/master/20200703101735.png)

```java
package com.wjw.limiting.mains;

public class Solution {

    public static void main(String[] ags) {

        int a = 100;
        int b = 100;

        System.out.println(a+b);
    }
}
```

```java
警告: 二进制文件Solution包含com.wjw.limiting.mains.Solution
        Classfile /Users/wangjiawei/Documents/project/limiting/target/classes/com/wjw/limiting/mains/Solution.class
Last modified 2020-5-14; size 615 bytes
        MD5 checksum c026bc46eab33943927b436ab54cbdad
        Compiled from "Solution.java"
public class com.wjw.limiting.mains.Solution
        minor version: 0
        major version: 52
        flags: ACC_PUBLIC, ACC_SUPER
        Constant pool:
        #1 = Methodref          #5.#23         // java/lang/Object."<init>":()V
        #2 = Fieldref           #24.#25        // java/lang/System.out:Ljava/io/PrintStream;
        #3 = Methodref          #26.#27        // java/io/PrintStream.println:(I)V
        #4 = Class              #28            // com/wjw/limiting/mains/Solution
        #5 = Class              #29            // java/lang/Object
        #6 = Utf8               <init>
   #7 = Utf8               ()V
           #8 = Utf8               Code
           #9 = Utf8               LineNumberTable
           #10 = Utf8               LocalVariableTable
           #11 = Utf8               this
           #12 = Utf8               Lcom/wjw/limiting/mains/Solution;
           #13 = Utf8               main
           #14 = Utf8               ([Ljava/lang/String;)V
           #15 = Utf8               ags
           #16 = Utf8               [Ljava/lang/String;
           #17 = Utf8               a
           #18 = Utf8               I
           #19 = Utf8               b
           #20 = Utf8               MethodParameters
           #21 = Utf8               SourceFile
           #22 = Utf8               Solution.java
           #23 = NameAndType        #6:#7          // "<init>":()V
           #24 = Class              #30            // java/lang/System
           #25 = NameAndType        #31:#32        // out:Ljava/io/PrintStream;
           #26 = Class              #33            // java/io/PrintStream
           #27 = NameAndType        #34:#35        // println:(I)V
           #28 = Utf8               com/wjw/limiting/mains/Solution
           #29 = Utf8               java/lang/Object
           #30 = Utf8               java/lang/System
           #31 = Utf8               out
           #32 = Utf8               Ljava/io/PrintStream;
           #33 = Utf8               java/io/PrintStream
           #34 = Utf8               println
           #35 = Utf8               (I)V
           {
public com.wjw.limiting.mains.Solution();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
        stack=1, locals=1, args_size=1
        0: aload_0
        1: invokespecial #1                  // Method java/lang/Object."<init>":()V
        4: return
        LineNumberTable:
        line 3: 0
        LocalVariableTable:
        Start  Length  Slot  Name   Signature
        0       5     0  this   Lcom/wjw/limiting/mains/Solution;

public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
        stack=3, locals=3, args_size=1
        0: bipush        100  // 注释: 将byte类型的常量100加载到操作数栈
        2: istore_1       //注释：将100加载到本地变量表的slot1
        3: bipush        100 //注释 将byte类型的变量100加载到操作数栈
        5: istore_2           //注释 将100加载到本地变量表的slot2
        6: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream; 获取PrintStream静态字段
        9: iload_1 	//将本地变量表的slot1加载到操作数栈
        10: iload_2 	//将本地变量表的slot2加载到操作数栈
        11: iadd 	//进行相加操作
        12: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V  调用PrintStream类中的println方法,参数类型为int,无返回值
        15: return
        LineNumberTable:
        line 7: 0
        line 8: 3
        line 10: 6
        line 11: 15
        LocalVariableTable:
        Start  Length  Slot  Name   Signature
        0      16     0   ags   [Ljava/lang/String;
        3      13     1     a   I
        6      10     2     b   I
        MethodParameters:
        Name                           Flags
        ags
        }
        SourceFile: "Solution.java"
```



#### 动态链接

##### 基本概念

每个栈桢都会包含一个执行运行时常量池中该栈桢所属方法的引用

##### 扩展

#### 方法出口

当方法执行完之后,只会有两种方法可以退回方法

1.正常退出:执行引擎遇到任意一个方法返回的字节码指令,这时会传递给上层的方法调用者，是否有返回值和返回值的类型是根据方法的返回追指令决定

2.异常退出:无论是Java虚拟机内部产生的异常还是代码中throw出的异常,只要在本方法的异常表中没有搜索到匹配的异常处理器,都会导致方法退出,这种方法的退出方式是不会给上层调用者任何返回值的

### 异常类型

##### StackOverflowError



``` java
package com.wjw.limiting.mains;

/**
 * @author wangjiawei
 * 模拟Java栈溢出
 */
public class JvmStack {

    //记录方法被调用的次数
    private int invokeMethodCount = 0;


    //递归调用
    public void stackLeak(){

        invokeMethodCount++;

        stackLeak();
    }

    public static void main(String[] args){

        JvmStack jvmStack = new JvmStack();

        try {

            jvmStack.stackLeak();

        }catch (Throwable e){

            System.out.println("invokeMethodCount:["+jvmStack.getInvokeMethodCount()+"]");

            System.out.println("stackTraceLength:["+Thread.currentThread().getStackTrace().length+"]");

            throw e;
        }
    }

    public int getInvokeMethodCount() {
        return invokeMethodCount;
    }
```

```java
invokeMethodCount:[18235]
        stackTraceLength:[2]
        Exception in thread "main" java.lang.StackOverflowError
        at com.wjw.limiting.mains.JvmStack.stackLeak(JvmStack.java:18)
        at com.wjw.limiting.mains.JvmStack.stackLeak(JvmStack.java:18)
        at com.wjw.limiting.mains.JvmStack.stackLeak(JvmStack.java:18)
        at com.wjw.limiting.mains.JvmStack.stackLeak(JvmStack.java:18)
        at com.wjw.limiting.mains.JvmStack.stackLeak(JvmStack.java:18)
```

##### OutOfMemoryError

未模拟成功

#### 面试题

#### 问题一:什么时候会发生栈溢出

栈是线程私有的,生命周期和线程保持一致,每个方法在执行的时候会创建栈桢,执行过程对应着栈桢的入栈与出栈,栈桢包含局部变量表,操作数栈,方法出口，动态链接等

如果线程请求的栈深度大于JVM允许的栈深度,会抛出StackOverflowError异常

如果java虚拟机栈内存允许动态扩展,同时申请不到足够的内存时,会抛出OutOfMemoryError异常

## 堆

### 基本概念

是所有线程共享的一块内存区域,负责存放对象实例和数组,不是所有的对象都会在堆上分配,有些是直接在栈上分配的

### 内存结构

![Java堆运行时内存结构](https://raw.githubusercontent.com/fontStep/photos/master/20200703101756.png)

年轻代: 主要用来存放新生的对象,一般占整个堆内存的1/3,年轻代又分为eden区,from区,to区,采用的GC算法为复制算法

Eden区：新对象主要的出生地,如果新创建的对象占用的内存很大,会直接分配到老年代,占整个新生代的8/10,触发的GC类型为MinorGC

ServivorFrom区: 上一次GC的幸存者,作为这一次GC的被扫描者

ServivorTo区：保留了一次GC过程中的幸存者

同时在GC的过程中ServivorFrom区和ServivorTo区会交换角色,分别作为幸存者的保留区,和被GC的区域

老年代：主要存放生命周期长的内存对象,采用的GC算法为标记清除算法,采用的GC类型为MajorGC,同时在进行MajorGC前都会触发一次MinorGC

### 对象的内存布局

主要包括对象头，实例数据，对齐填充这三块区域

![java对象布局](https://raw.githubusercontent.com/fontStep/photos/master/20200703101814.png)



#### 压缩指针

![java对象头中一般对象指针](https://raw.githubusercontent.com/fontStep/photos/master/20200703101839.png)

理解

```java
<!-- https://mvnrepository.com/artifact/org.openjdk.jol/jol-core -->
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

例子一：空对象的情况

```java
package com.wjw.limiting.mains;


import org.openjdk.jol.info.ClassLayout;

public class T01 {

/*    byte a = 0;

    short b = 1;

    int c = 1;

    long d = 1;*/

    //Object a;

    //String string = "123";
    public static void main(String[] args) {

        T01 t = new T01();

        System.out.println(ClassLayout.parseInstance(t).toPrintable());
    }
}
```

开启指针压缩的情况的对象布局情况

```java
com.wjw.limiting.mains.T01 object internals:
        OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
        0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4        (object header)                           54 c3 00 f8 (01010100 11000011 00000000 11111000) (-134167724)
        12     4        (loss due to the next object alignment)
        Instance size: 16 bytes
        Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

12k(对象头)+4k(对齐填充) = 16k

关闭指针压缩的对象布局情况

```java
com.wjw.limiting.mains.T01 object internals:
        OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
        0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4        (object header)                           28 f0 ff 1a (00101000 11110000 11111111 00011010) (452980776)
        12     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        Instance size: 16 bytes
        Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

16k(对象头)

例子二：基本数据类型

```java
package com.wjw.limiting.mains;


import org.openjdk.jol.info.ClassLayout;

public class T01 {


    int a = 0;

    public static void main(String[] args) {

        T01 t = new T01();

        System.out.println(ClassLayout.parseInstance(t).toPrintable());
    }
}
```

```java
com.wjw.limiting.mains.T01 object internals:
        OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
        0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4        (object header)                           05 c1 00 f8 (00000101 11000001 00000000 11111000) (-134168315)
        12     4    int T01.a                                     0
        Instance size: 16 bytes
        Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

12k(对象头)+4k(实例数据) = 16k

```java
OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
        0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4        (object header)                           28 40 f9 0e (00101000 01000000 11111001 00001110) (251215912)
        12     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        16     4    int T01.a                                     0
        20     4        (loss due to the next object alignment)
```

16k(对象头)+4k(实例数据) +4k(对齐)=24k

例子三:普通引用类型

```java
package com.wjw.limiting.mains;


import org.openjdk.jol.info.ClassLayout;

public class T01 {


    Object o;

    public static void main(String[] args) {

        T01 t = new T01();

        System.out.println(ClassLayout.parseInstance(t).toPrintable());
    }
}
```

```java
com.wjw.limiting.mains.T01 object internals:
        OFFSET  SIZE               TYPE DESCRIPTION                               VALUE
        0     4                    (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4                    (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4                    (object header)                           05 c1 00 f8 (00000101 11000001 00000000 11111000) (-134168315)
        12     4   java.lang.Object T01.o                                     null
        Instance size: 16 bytes
        Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

12k(对象头)+4k(实例数据) =16k

```java
com.wjw.limiting.mains.T01 object internals:
        OFFSET  SIZE               TYPE DESCRIPTION                               VALUE
        0     4                    (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4                    (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4                    (object header)                           28 90 f5 1d (00101000 10010000 11110101 00011101) (502632488)
        12     4                    (object header)                           02 00 00 00 (00000010 00000000 00000000 00000000) (2)
        16     8   java.lang.Object T01.o                                     null
        Instance size: 24 bytes
        Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

16k(对象头)+8k(实例数据) = 24k

结论：开启对象指针压缩的情况下,对象实例数据中的引用类型,对象头中的Mark Word,对象头中类指针会被压缩,实例数据中的基本数据类型不会产生变化

#### 对象头

对象头主要包括Mark Word，指向类的指针，数组长度（只有数组对象才有）,在开启指针压缩的情况下占8k

##### mark word

Mark word记录了对象和锁有关的信息,32bit的情况下

![markword结构](https://raw.githubusercontent.com/fontStep/photos/master/20200703101858.png)

##### synchronized锁的升级过程

无锁状态下,Mark Word记录对象的HashCode，锁标志位是01，是否偏向锁为0

当对象作为同步锁时,线程A获取锁资源时,同时没有其他线程争抢锁资源时,线程A会获取偏向锁,是否偏向锁的标志位为1,锁标志为01,同时前23bit位记录线程ID

当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步锁的代码

当线程B试图获取锁资源时,JVM发现同步锁处理偏向状态,但是Mark Word中的线程ID记录的不是B,那么线程会先用CAS操作试图获取锁，如果线程A释放锁，那么线程B抢锁成功,就把Mark Word里的线程ID修改为线程B的ID，代表线程B获得了这个偏向锁，如果抢锁失败,会进行锁升级操作,将锁升级为轻量级锁

偏向锁状态抢锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是CAS操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步锁代码。如果保存失败，表示抢锁失败，竞争太激烈

轻量级锁抢锁失败，JVM会使用自旋锁，自旋锁不是一个锁状态，只是代表不断的重试，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步锁代码

轻量级锁抢锁失败，JVM会使用自旋锁，自旋锁不是一个锁状态，只是代表不断的重试，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步锁代码

### 对象堆上分配

#### 分配在eden区

大部分情况下,对象在新生代Eden区中分配,当Eden区没有足够空间进行分配时,虚拟机将发起一次Minor GC

#### 大对象分配在old区

大对象指的是需要大量连续内存空间的Java对象

可是通过设置JVM参数XX:PretenureSizeThreshold 来让对象直接在old区分配内存,默认值为0,表示不管对象的大小都会优先在eden区分配

查看JVM参数默认值指令

**-XX:+PrintFlagsInitial参数 ** 显示所有可设置参数及默认值

**-XX:+PrintFlagsFinal参数** 可以获取到所有可设置参数及值(手动设置之后的值)

**注意： PretenureSizeThreshold参数只会对Serial和ParNew两款年轻代收集器有效**

#### 长期存活的对象进入老年代

分代年龄：对象每次经过一次Minior GC之后对应的分代年龄会在之前的基础上加一,当年龄增加到一定程度（默认为15岁）可以通过MaxTenuringThreshold参数进行设置,**分代年龄的大小存储在对象头的Mark Word字段占4bit,最大只能设置为15**

#### 老年代空间分配担保

在发生Minor GC之前,虚拟机会检查**老年代最大可用的连续空间是否大于新生代所有对象的总空间**

如果大于,则此次Minor GC是安全的

如果小于,则虚拟机会查看**HandlePromotionFailure**设置值是否允许担保失败。
 如果HandlePromotionFailure=true，那么会**继续检查老年代最大可用连续空间是否大于历次晋升到老年代的对象的平均大小**，如果大于，则尝试进行一次Minor GC，但这次Minor GC依然是有风险的；如果小于或者HandlePromotionFailure=false，则改为进行一次Full GC

上面提到了Minor GC依然会有风险，是因为新生代采用**复制收集算法**，假如大量对象在Minor GC后仍然存活（最极端情况为内存回收后新生代中所有对象均存活），而Survivor空间是比较小的，这时就需要老年代进行分配担保，把Survivor无法容纳的对象放到老年代。**老年代要进行空间分配担保，前提是老年代得有足够空间来容纳这些对象**，但一共有多少对象在内存回收后存活下来是不可预知的，**因此只好取之前每次垃圾回收后晋升到老年代的对象大小的平均值作为参考**。使用这个平均值与老年代剩余空间进行比较，来决定是否进行Full GC来让老年代腾出更多空间。

取平均值仍然是一种**概率性的事件**，如果某次Minor GC后存活对象陡增，远高于平均值的话，必然导致担保失败，如果出现了分配担保失败，**就只能在失败后重新发起一次Full GC**。虽然存在发生这种情况的概率，但**大部分时候都是能够成功分配担保**的，这样就避免了过于频繁执行Full GC。

### 异常类型

#### OutOfMemoryError

产生错误的原因:一直创建对象

```java
package com.wjw.limiting.mains;


import java.util.ArrayList;
import java.util.List;

public class T01 {


    /**
     * 
     * @param args
     */
    public static void main(String[] args) {

        List list = new ArrayList();

        while (true){

            list.add(new Object());
        }
        
    }
}
```

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        Disconnected from the target VM, address: '127.0.0.1:59472', transport: 'socket'
        at java.util.Arrays.copyOf(Arrays.java:3210)
        at java.util.Arrays.copyOf(Arrays.java:3181)
        at java.util.ArrayList.grow(ArrayList.java:265)
        at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
        at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
        at java.util.ArrayList.add(ArrayList.java:462)
        at com.wjw.limiting.mains.T01.main(T01.java:21)
```

## 本地方法栈

本地方法栈与虚拟机栈发挥的作用是相似的,它们之间的区别不过是虚拟机为虚拟机执行java方法,然而本地方法栈则为虚拟机使用native方法

## 方法区

方法区是所有线程共享的内存区域,用于存储虚拟机加载的类信息，常量信息,静态变量

## 直接内存

# 对象已死

## 可达性分析

可达性分析是通过“GC Roots“的对象作为起始点,从这些节点开始向下搜索，搜索所走过的所有路径称为引用链,当一个对象到GC Roots没有任何引用链相连时,则证明这些此对象是不可用,对象将会被判断为可回收对象

可以作为GC Roots的区域为

1. Java虚拟机栈中本地变量表中引用的对象
2. 方法区中静态属性引用的对象
3. 方法区中常量引用的对象
4. 本地栈中引用的对象

# 垃圾收集算法

## 标记清除算法

首先标记所有需要被回收的对象,在标记完成之后，统一回收被标记的对象

缺点：容易产生内存碎片,导致较大对象无法在堆空间分配,从而触发minior GC来进行垃圾回收操作

## 复制算法

将可用的内存空间划分为大小相同的两块内存空间，当一块内存空间快满时,将这块空间还存活的对象移动到另一块内存空间

缺点：会导致一半的内存空间浪费

## 标记整理算法

首先标记需要被收回的对象,在标记完成之后，将存活的对象移动到一端

# 垃圾收集器

# 类加载机制

## 类加载的时机

类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括加载，验证，准备，解析，使用和卸载7个阶段

什么情况下需要开始类加载过程的第一个阶段

1) 遇到new，getstatic，putstatic或invokestatic这4条字节码指令时，如果当前类还没有被初始化，则需要先粗发初始化,这4条指令分别对应的场景为使用new关键字实例化对象,读取，设置类变量时，调用静态方法时

2）对类进行反射调用的时候，如果该类没有被初始化，则需要先触发对该类进行初始化

3）初始化一个类的时候，如果发现其父类还没有进行初始化，则会对父类进行初始化

## 类加载过程

### 加载

加载是将类加载到内存中的一个阶段

1）通过类的全限定名和获取定义此类的二进制字节流

2）将这个字节流所代表的讲台存储结构转化为方法区的运行时数据结构

3）在内存中生成一个代表这个类的Java.lang.Class对象，作为方法去这个类的各种数据的访问入口

## 验证

验证阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，不会危害虚拟机自身的安全

1.文件格式的验证

验证字节流是否符合Class文件格式的规范,并且能被当前虚拟机所处理

2.元数据的验证

对元数据进行语义分析,保证符合Java虚拟机的规范

3.字节码验证

通过数据流和控制流分析，确定程序语义是合法的,符合逻辑的，这个阶段会对类的方法进行校验分析，保证被校验类的方法在运行的过程中不会做出危害虚拟机安全的事件

4.符合引用验证

## 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，设置的初始值不是Java代码中赋值的值，而是Java基本类型对应的零值

被final关键字修饰的类变量被设置为Java代码中设置的值

## 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程

## 初始化



# JVM命令

## jps

JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。

### 命令格式

jps [options] [hostid]

optinos参数

- -l : 输出主类全名或jar路径
- -q : 只输出LVMID
- -m : 输出JVM启动时传递给main()的参数
- -v : 输出JVM启动时显示指定的JVM参数

## jstat

jstat(JVM statistics Monitoring)是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据

## 命令格式

jstat [option] LVMID [interval] [count]

参数

- [option] : 操作参数

- LVMID : 本地虚拟机进程ID

- [interval] : 连续输出的时间间隔

- [count] : 连续输出的次数

  

option参数总览

| **Option**       | **Displays…**                                                |
| ---------------- | ------------------------------------------------------------ |
| class            | class loader的行为统计                                       |
| compiler         | HostSpt JIT编译器行为统计                                    |
| gc               | 垃圾回收堆得行为统计                                         |
| gccapacity       | 各个垃圾回收代容量和他们对应的空间统计                       |
| gcutil           | 垃圾回收统计概述                                             |
| gccause          | 垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因 |
| gcnew            | 新生代行为统计                                               |
| gcnewcapacity    | 新生代与其相应的内存空间的统计                               |
| gcold            | 年老代和永生代行为统计                                       |
| gcoldcapacity    | 年老代行为统计                                               |
| gcpermcapacity   | 永生代行为统计                                               |
| printcompilation | HotSpot编译方法统计                                          |

#### option 参数详解

##### -class

监视类装载、卸载数量、总空间以及耗费的时间

```xml
$ jstat -class 11589
 Loaded  Bytes  Unloaded  Bytes     Time   
  7035  14506.3     0     0.0       3.67
```

> - Loaded : 加载class的数量
> - Bytes : class字节大小
> - Unloaded : 未加载class的数量
> - Bytes : 未加载class的字节大小
> - Time : 加载时间

##### -compiler

输出JIT编译过的方法数量耗时等

```xml
$ jstat -compiler 1262
Compiled Failed Invalid   Time   FailedType FailedMethod
    2573      1       0    47.60          1 org/apache/catalina/loader/WebappClassLoader findResourceInternal  
```

> - Compiled : 编译数量
> - Failed : 编译失败数量
> - Invalid : 无效数量
> - Time : 编译耗时
> - FailedType : 失败类型
> - FailedMethod : 失败方法的全限定名

##### -gc

垃圾回收堆的行为统计，**常用命令**

```xml
$ jstat -gc 1262
 S0C    S1C     S0U     S1U   EC       EU        OC         OU        PC       PU         YGC    YGCT    FGC    FGCT     GCT   
26112.0 24064.0 6562.5  0.0   564224.0 76274.5   434176.0   388518.3  524288.0 42724.7    320    6.417   1      0.398    6.815
```

**C即Capacity 总容量，U即Used 已使用的容量**

> - S0C : survivor0区的总容量
> - S1C : survivor1区的总容量
> - S0U : survivor0区已使用的容量
> - S1C : survivor1区已使用的容量
> - EC : Eden区的总容量
> - EU : Eden区已使用的容量
> - OC : Old区的总容量
> - OU : Old区已使用的容量
> - PC 当前perm的容量 (KB)
> - PU perm的使用 (KB)
> - YGC : 新生代垃圾回收次数
> - YGCT : 新生代垃圾回收时间
> - FGC : 老年代垃圾回收次数
> - FGCT : 老年代垃圾回收时间
> - GCT : 垃圾回收总消耗时间

```xml
$ jstat -gc 1262 2000 20
```

这个命令意思就是每隔2000ms输出1262的gc情况，一共输出20次

##### -gccapacity

同-gc，不过还会输出Java堆各区域使用到的最大、最小空间

```xml
$ jstat -gccapacity 1262
 NGCMN    NGCMX     NGC    S0C   S1C       EC         OGCMN      OGCMX      OGC        OC       PGCMN    PGCMX     PGC      PC         YGC    FGC 
614400.0 614400.0 614400.0 26112.0 24064.0 564224.0   434176.0   434176.0   434176.0   434176.0 524288.0 1048576.0 524288.0 524288.0    320     1  
```

> - NGCMN : 新生代占用的最小空间
> - NGCMX : 新生代占用的最大空间
> - OGCMN : 老年代占用的最小空间
> - OGCMX : 老年代占用的最大空间
> - OGC：当前年老代的容量 (KB)
> - OC：当前年老代的空间 (KB)
> - PGCMN : perm占用的最小空间
> - PGCMX : perm占用的最大空间

##### -gcutil

同-gc，不过输出的是已使用空间占总空间的百分比

```xml
$ jstat -gcutil 28920
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
 12.45   0.00  33.85   0.00   4.44  4       0.242     0    0.000    0.242
```

##### -gccause

垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因

```xml
$ jstat -gccause 28920
  S0     S1     E      O      P       YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
 12.45   0.00  33.85   0.00   4.44      4    0.242     0    0.000    0.242   Allocation Failure   No GC  
```

> - LGCC：最近垃圾回收的原因
> - GCC：当前垃圾回收的原因

##### -gcnew

统计新生代的行为

```xml
$ jstat -gcnew 28920
 S0C      S1C      S0U        S1U  TT  MTT  DSS      EC        EU         YGC     YGCT  
 419392.0 419392.0 52231.8    0.0  6   6    209696.0 3355520.0 1172246.0  4       0.242
```

> - TT：Tenuring threshold(提升阈值)
> - MTT：最大的tenuring threshold
> - DSS：survivor区域大小 (KB)

##### -gcnewcapacity

新生代与其相应的内存空间的统计

```xml
$ jstat -gcnewcapacity 28920
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC        YGC   FGC 
 4194304.0  4194304.0  4194304.0 419392.0 419392.0 419392.0 419392.0  3355520.0  3355520.0     4     0
```

> - NGC:当前年轻代的容量 (KB)
> - S0CMX:最大的S0空间 (KB)
> - S0C:当前S0空间 (KB)
> - ECMX:最大eden空间 (KB)
> - EC:当前eden空间 (KB)

##### -gcold

统计旧生代的行为

```java
$ jstat -gcold 28920
   PC       PU        OC           OU       YGC    FGC    FGCT     GCT   
1048576.0  46561.7   6291456.0     0.0      4      0      0.000    0.242
```

##### -gcoldcapacity

统计旧生代的大小和空间

```java
$ jstat -gcoldcapacity 28920
   OGCMN       OGCMX        OGC         OC         YGC   FGC    FGCT     GCT   
  6291456.0   6291456.0   6291456.0   6291456.0     4     0    0.000    0.242
```

##### -gcpermcapacity

永生代行为统计

```java
$ jstat -gcpermcapacity 28920
    PGCMN      PGCMX       PGC         PC      YGC   FGC    FGCT     GCT   
 1048576.0  2097152.0  1048576.0  1048576.0     4     0    0.000    0.242
```

##### -printcompilation

hotspot编译方法统计

```java
$ jstat -printcompilation 28920
    Compiled  Size  Type Method
    1291      78     1    java/util/ArrayList indexOf
```

> - Compiled：被执行的编译任务的数量
> - Size：方法字节码的字节数
> - Type：编译类型
> - Method：编译方法的类名和方法名。类名使用"/" 代替 "." 作为空间分隔符. 方法名是给出类的方法名. 格式是一致于HotSpot - XX:+PrintComplation 选项



## jstack

jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

### 命令格式

```java
jstack [option] LVMID
```

### option参数

> - -F : 当正常输出请求不被响应时，强制输出线程堆栈
> - -l : 除堆栈外，显示关于锁的附加信息
> - -m : 如果调用到本地方法的话，可以显示C/C++的堆栈

### 示例

```java
$ jstack -l 11494|more
2016-07-28 13:40:04
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.71-b01 mixed mode):

"Attach Listener" daemon prio=10 tid=0x00007febb0002000 nid=0x6b6f waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"http-bio-8005-exec-2" daemon prio=10 tid=0x00007feb94028000 nid=0x7b8c waiting on condition [0x00007fea8f56e000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000cae09b80> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
      .....
```

### 分析

这里有一篇文章解释的很好
[分析打印出的文件内容](http://www.hollischuang.com/archives/110)

## jinfo

jinfo(JVM Configuration info)这个命令作用是实时查看和调整虚拟机运行参数。
之前的jps -v口令只能查看到显示指定的参数，如果想要查看未被显示指定的参数的值就要使用jinfo口令

### 命令格式

```java
jinfo [option] [args] LVMID
```

### option参数

> - -flag : 输出指定args参数的值
> - -flags : 不需要args参数，输出所有JVM参数的值
> - -sysprops : 输出系统属性，等同于System.getProperties()

### 示例

```java
$ jinfo -flag 11494
-XX:CMSInitiatingOccupancyFraction=80
```



## jmap

jmap(JVM Memory Map)命令用于生成heap dump文件，如果不使用这个命令，还阔以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件。
jmap不仅能生成dump文件，还阔以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

### 命令格式

```xml
jmap [option] LVMID
```

### option参数

> - dump : 生成堆转储快照
> - finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
> - heap : 显示Java堆详细信息
> - histo : 显示堆中对象的统计信息
> - permstat : to print permanent generation statistics
> - F : 当-dump没有响应时，强制生成dump快照

### 示例

##### -dump

常用格式

```xml
-dump::live,format=b,file=<filename> pid 
```

dump堆到文件,format指定输出格式，live指明是活着的对象,file指定文件名

```xml
$ jmap -dump:live,format=b,file=dump.hprof 28920
  Dumping heap to /home/xxx/dump.hprof ...
  Heap dump file created
```

dump.hprof这个后缀是为了后续可以直接用MAT(Memory Anlysis Tool)打开。

##### -finalizerinfo

打印等待回收对象的信息

```xml
$ jmap -finalizerinfo 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01
  Number of objects pending for finalization: 0
```

可以看到当前F-QUEUE队列中并没有等待Finalizer线程执行finalizer方法的对象。

##### -heap

打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况,可以用此来判断内存目前的使用情况以及垃圾回收情况

```java
$ jmap -heap 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01  

  using thread-local object allocation.
  Parallel GC with 4 thread(s)//GC 方式  

  Heap Configuration: //堆内存初始化配置
     MinHeapFreeRatio = 0 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
     MaxHeapFreeRatio = 100 //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
     MaxHeapSize      = 2082471936 (1986.0MB) //对应jvm启动参数-XX:MaxHeapSize=设置JVM堆的最大大小
     NewSize          = 1310720 (1.25MB)//对应jvm启动参数-XX:NewSize=设置JVM堆的‘新生代’的默认大小
     MaxNewSize       = 17592186044415 MB//对应jvm启动参数-XX:MaxNewSize=设置JVM堆的‘新生代’的最大大小
     OldSize          = 5439488 (5.1875MB)//对应jvm启动参数-XX:OldSize=<value>:设置JVM堆的‘老生代’的大小
     NewRatio         = 2 //对应jvm启动参数-XX:NewRatio=:‘新生代’和‘老生代’的大小比率
     SurvivorRatio    = 8 //对应jvm启动参数-XX:SurvivorRatio=设置年轻代中Eden区与Survivor区的大小比值 
     PermSize         = 21757952 (20.75MB)  //对应jvm启动参数-XX:PermSize=<value>:设置JVM堆的‘永生代’的初始大小
     MaxPermSize      = 85983232 (82.0MB)//对应jvm启动参数-XX:MaxPermSize=<value>:设置JVM堆的‘永生代’的最大大小
     G1HeapRegionSize = 0 (0.0MB)  

  Heap Usage://堆内存使用情况
  PS Young Generation
  Eden Space://Eden区内存分布
     capacity = 33030144 (31.5MB)//Eden区总容量
     used     = 1524040 (1.4534378051757812MB)  //Eden区已使用
     free     = 31506104 (30.04656219482422MB)  //Eden区剩余容量
     4.614088270399305% used //Eden区使用比率
  From Space:  //其中一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  To Space:  //另一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  PS Old Generation //当前的Old区内存分布
     capacity = 86507520 (82.5MB)
     used     = 0 (0.0MB)
     free     = 86507520 (82.5MB)
     0.0% used
  PS Perm Generation//当前的 “永生代” 内存分布
     capacity = 22020096 (21.0MB)
     used     = 2496528 (2.3808746337890625MB)
     free     = 19523568 (18.619125366210938MB)
     11.337498256138392% used  

  670 interned Strings occupying 43720 bytes.
```

可以很清楚的看到Java堆中各个区域目前的情况。

##### -histo

打印堆的对象统计，包括对象数、内存大小等等 （因为在dump:live前会进行full gc，如果带上live则只统计活对象，因此不加live的堆大小要大于加live堆的大小 ）

```xml
$ jmap -histo:live 28920 | more
 num     #instances         #bytes  class name
----------------------------------------------
   1:         83613       12012248  <constMethodKlass>
   2:         23868       11450280  [B
   3:         83613       10716064  <methodKlass>
   4:         76287       10412128  [C
   5:          8227        9021176  <constantPoolKlass>
   6:          8227        5830256  <instanceKlassKlass>
   7:          7031        5156480  <constantPoolCacheKlass>
   8:         73627        1767048  java.lang.String
   9:          2260        1348848  <methodDataKlass>
  10:          8856         849296  java.lang.Class
  ....
```

仅仅打印了前10行

`xml class name`是对象类型，说明如下：

```xml
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象
```

##### -permstat

打印Java堆内存的永久保存区域的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。

```xml
$ jmap -permstat 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01
  finding class loader instances ..done.
  computing per loader stat ..done.
  please wait.. computing liveness.liveness analysis may be inaccurate ...
  
  class_loader            classes bytes   parent_loader           alive?  type  
  <bootstrap>             3111    18154296          null          live    <internal>
  0x0000000600905cf8      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006008fcb48      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006016db798      0       0       0x00000006008d3fc0      dead    java/util/ResourceBundle$RBClassLoader@0x0000000780626ec0
  0x00000006008d6810      1       3056      null          dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
```

##### -F

强制模式。如果指定的pid没有响应，请使用jmap -dump或jmap -histo选项。此模式下，不支持live子选项。























