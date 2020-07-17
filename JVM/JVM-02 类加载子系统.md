# JVM-02 类加载子系统

**JVM总体架构图**

![](http://picturebed.yifelix.cn/blogimgs/jvm02/01.png)

本文针对Class Loader SubSystem（类加载器子系统）进行讲解工作流程

## 一、类加载子系统的作用

- 类加载器子系统负责从文件系统或者网络中加载class文件，class文件在文件开头有特定的文件标识。
- ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。
- 加载的类信息存放在一块称为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包含字符串常量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）



## 二、类加载的过程

![](http://picturebed.yifelix.cn/blogimgs/jvm02/02.png)

**分为三个阶段：加载（Loading）、链接（Linking）、初始化（Initization）**

**加载(Loading):**

- 通过一个类的全限定明获取定义此类的二进制字节流
- 将这个字节流所代表的的静态存储结构转化为方法区的运行时数据
- 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口



**链接(Linking)，分为三块，验证、准备、解析：**

1. 验证
   - 目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全
   - 主要包括四种验证，文件格式验证，源数据验证，字节码验证，符号引用验证
2. 准备
   - 为类变量分配内存并且设置该类变量的默认初始值，即零值
   - 不包含用final修饰的sttic，因为final在编译的时候就会分配了，准备阶段会显式初始化
   - 不会为实例变量分配初始化，类变量会分配在方法去中，而实例变量是会随着对象一起分配到java堆中
3. 解析
   - 将常量池内的符号引用转换为直接引用的过程
   - 事实上，解析操作网晚会伴随着jvm在执行完初始化之后再执行
   - 符号引用就是一组符号来描述所引用的目标。符号应用的字面量形式明确定义在《java虚拟机规范》的class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄
   - 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的CONSTANT_Class_info/CONSTANT_Fieldref_info、CONSTANT_Methodref_info等。

**初始化(Initization)：**

- 初始化阶段就是执行类构造器方法<clinit>()的过程（注：clinit即“class or interface initialization method”，注意他并不是指构造器init()）
- 这个方法不需要定义，是javac编译器自动收集类中所有类变量的赋值动作和静态代码块中的语句合并而来。如果没有静态变量的赋值以及没有静态代码块，就不会有<cinit>()方法
- 构造器方法中指令按语句在源文件中出现的顺序执行
- <cinit>()不同于类的构造器
- 若该类具有父类，JVM会保证子类的<clinit>()执行前，父类的<clinit>()已经执行完毕。
- 虚拟机必须保证一个类的<cinit>()方法在多线程下被同步加锁。



下面看一个例子

````java
/**
* 类的初始化
*/
public class ClassInitTest {
    private static int num = 20;

    static {
        num = 2;
        number = 1;
    }

    private static int number = 2;

    public static void main(String[] args) {
        System.out.println(ClassInitTest.num);
        System.out.println(ClassInitTest.number);
    }
}
````

![](http://picturebed.yifelix.cn/blogimgs/jvm02/03.png)

此图使用的IDEA安装的**jclasslib Bytecode viewer**，进行字节码文件反编译出来的，先看num先被赋值为20，后在被赋值成2，而在看number声明在静态代码块之后，但是没有报错，是因为类在加载的过程中，是先分配空间，在进行初始化，number在分配空间是0到静态代码块中赋值成1在被赋值成2，所以最后是0--->1--->2。

## 三、类加载器分类

- JVM支持两种类型的加载器，分别为**引导类加载器C/C++实现（BootStrap ClassLoader）和自定义类加载器由Java实现**

- 从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是java虚拟机规范却没有这么定义，而是**将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器**。

![](http://picturebed.yifelix.cn/blogimgs/jvm02/04.png)

**上图中所有的类加载器都是包含关系，而不是继承关系**

在程序中我们最常见的类加载器是：**引导类加载器BootStrapClassLoader、自定义类加载器(Extension Class Loader、System Class Loader、User-Defined ClassLoader）**

### 3.1 启动类加载器（引导类加载器，Bootstrap ClassLoader）

- 使用C/C++语言实现的，嵌套在JVM内部
- 用来加载Java的核心类库（JAVA_HOME/jre/lib/rt.jar、resource.jar或者sun.boot.class.path路径下的内容），用于提供JVM自身需要的类。
- 因为是C/C++语言实现的，所以并不继承自java.lang.ClassLoader，没有父加载器
- 加载扩展类和应用程序类加载器，并指定为他们的父类加载器出于安全考虑，**Bootstrap ClassLoader**只加载包名为java、javax、sun等开头的类。

### 3.2扩展类加载器（Extension ClassLoader）

- Java 语言编写，由sun.misc.Launcher$ExtClassLoader实现
- 派生于ClassLoader类
- 父类加载器为ExtensionClassloader
- 它负责加载环境变量classpath或者系统属性，java.class.path指定路径下的类库
- 该类加载器是程序中默认的类加载器，一般来说，java应用类都是由他来加载完成的
- 通过ClassLoader.getSystemClassLoader()可以获取到该类加载器

### 3.3应用程序类加载器（系统类加载器，AppClassLoader）

- java语言编写， 由sun.misc.Launcher$AppClassLoader实现
- 派生于ClassLoader类
- 它负责加载环境变量classpath或系统属性 java.class.path指定路径下的类库
- **该类加载器是程序中默认的类加载器**，一般来说，java应用的类都是由它来完成加载
- 通过ClassLoader#getSystemClassLoader()方法可以获取到该类加载器

````java
public class ClassLoaderTest {

    public static void main(String[] args) {

        //获取系统类加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        //sun.misc.Launcher$AppClassLoader@18b4aac2
        System.out.println(systemClassLoader); 

        //获取其上层：扩展类加载器
        ClassLoader extClassLoader = systemClassLoader.getParent();
        //sun.misc.Launcher$ExtClassLoader@1b6d3586
        System.out.println(extClassLoader); 

        //扩展类加载器上层就是引导类加载器，但是是C++编写无法获取到返回null
        ClassLoader bootstrapLoader = extClassLoader.getParent();
        System.out.println(bootstrapLoader);

        //用户自定义类来说：默认使用系统类加载器进行加载
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        //sun.misc.Launcher$AppClassLoader@18b4aac2
        System.out.println(classLoader);  

        //String类使用引导类加载器进行加载的 ---> Java的核心类库都是使用引导类加载器进行加载的
        ClassLoader stringClassLoader = String.class.getClassLoader();
        System.out.println(stringClassLoader);  //null

    }
}
````



## 四、ClassLoader与常用方法

ClassLoader类是一个抽象类，其后所有的类加载器都继承自ClassLoader（不包括启动类加载器）

sun.misc.Launcher他相当于一个Java虚拟机的入口应用

![](http://picturebed.yifelix.cn/blogimgs/jvm02/05.png)

**常用方法:**

| 方法名称                                          | 描述                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| getParent()                                       | 返回该类加载器的超类加载器                                   |
| loadClass(String name)                            | 加载名称为name的类，返回结果为java.lang.Class类的实例        |
| findClass(String name)                            | 查找名称为name的类，返回结果为java.lang.Class类的实例        |
| findLoadedClass(String name)                      | 查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例 |
| defineClass(String name,byte[] b,int off,int len) | 把字节数组b中的内容转换为一个Java类，返回结果为java.lang.Class类的实例 |
| resolveClass(Class<?> c)                          | 连接指定的一个Java类                                         |

**获取ClassLoader的路径：**

![](http://picturebed.yifelix.cn/blogimgs/jvm02/07.png)

