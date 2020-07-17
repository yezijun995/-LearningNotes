# JVM-01 简介

## 一、什么是JVM

JVM是Java Virtual Machine(Java虚拟机的缩写)，是遵循Java虚拟机规范生产出来的虚拟机。它是一台执行Java字节码的虚拟计算器，拥有独立的运行机制，其运行的Java字节码也未必由Java语言编译而成。Java技术的核心就是Java虚拟机，因为所有的Java程序都运行在Java虚拟机内部。也是因为有JVM的存在，才能使得Java能**一次编译，到处运行。**

![](http://picturebed.yifelix.cn/blogimgs/jvm01/01.jpg)

 引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。



## 二、一些主流的JVM

- HotSpot VM：它是JDK默认内置的JVM，是目前最主流的VM，是Sum JDK和Open JDK所带的虚拟机。
- BEA JRockit：在1.6的时代Java还是Sun公司时，Oracle自己出了一个Java虚拟机，它是世界上最快的JVM
- IBM J9：是IBM用于自身产Java产品，也号称是世界上最快的Java虚拟机。
- Sun Classic VM：它是世界上第一款商用Java虚拟机。jdk1.4时完全被淘汰，现在HotSpot内置此虚拟机，此虚拟机 内部只提供了解释器。
- Exact VM：jdk1.2时Sun公司提供了此虚拟机。具有现代高性能虚拟机的雏形，热点探测与编译器与解释器混合工作模式
- AliJVM：阿里对JVM进行了定制，替换了Oracle官方JVM
- Universal GraalVM：是Oracle在2018年4月开源的一个实验性产品，真正意义上的跨语言全栈虚拟机，可以作为“任何语言”的运行平台使用

## 三、JVM位置

### 1.JVM在计算机中所处的位置

![](http://picturebed.yifelix.cn/blogimgs/jvm01/02.png)

### 2.Java架构

![](http://picturebed.yifelix.cn/blogimgs/jvm01/03.jpg)

## 四、JVM体系结构概述

![](http://picturebed.yifelix.cn/blogimgs/jvm01/04.jpg)

JVM虚拟机由上图三个子系统构成，分别是类加载子系统、JVM运行时数据区和执行引擎。

1. 类装载器子系统(Class Loader)：用来加载Java类到Java虚拟机中

2. 运行时数据区(Runtime Data Area)：运行数据区是整个JVM的重点。我们所有写的程序都被加载到这里，之后才开始运行

3. 执行引擎（Execution Engine）：负责解释命令，提交操作系统执行
4. 本地接口(Native Interface)：本地接口的作用是融合不同的编程语言为Java 所用，它的初衷是融合C/C++ 程序，此处不多做介绍。

## 五、指令集架构

Java编译器输入的指令流基本上是一种基于**栈的指令集架构**，另外一种指令集架构则是基于寄存器的指令集结构。

````asm
//基于栈的指令
iconst_1   //常量1入栈
iconst_2   //常量2入栈
iadd       //常量1/2出栈执行相加
istore_0   //结果入栈

//基于寄存器指令
mov eax，1	//将eax寄存器的值设为1
add eax，1	//将eax寄存器的值加1
````

### 1.栈的指令集架构

- 设计和实现更简单，适用于资源受限的系统
- 避开了寄存器的分配难题：使用零地址指令分配方式
- 指令流中的指令大部分是零地址指令，其执行过程依赖于操作栈。指令集更小，编译容易实现
- 不需要硬件支持，可一致性更好，更好实现跨平台

### 2.寄存器的指令集架构

- 典型的应用是x86的二进制指令集：比如传统的PC以及Android的Davlik虚拟机
- 指令集结构则完全依赖硬件，可移植性差
- 性能优秀和执行更高效
- 花费更少的指令去完成一项操作
- 在大部分情况下，基于寄存器结构的指令集往往都以一地址指令、二地址指令和三地址指令为主，而基于栈式架构的指令集却是以零地址指令为主



**二者主要区别：基于栈的指令集主要的优点就是可移植, 指令集小，指令多；缺点是执行速度慢,相同操作指令数要多很多。而寄存器指令会少很多。**

## 六、JVM生命周期

**1. JVM实例诞生**

Java虚拟机的诞生是由引导类加载器(bootstrap class loader)创建一个初始类(initial class)来完成的，这个类是由虚拟机的具体实现指定的。

**2. 虚拟机的执行**

- 一个运行中的java虚拟机有着一个清晰的任务：执行Java程序；
- 程序开始执行的时候他才运行，程序结束时他就停止；
- 执行一个所谓的Java程序的时候，真真正正在执行的是一个叫做Java虚拟机的进程。

**3. JVM的消亡**

- 程序正常执行结束
- 程序异常或错误而异常终止
- 操作系统错误导致终止
- 某线程调用Runtime类或System类的exit方法，或Runtime类的halt方法，并且java安全管理器也允许这次exit或halt操作
- 除此之外，JNI规范描述了用JNI Invocation API来加载或卸载Java虚拟机时，Java虚拟机的退出情况