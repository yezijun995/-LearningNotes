# Java NIO

## 一、NIO概述

### NIO简介

​		**Java NIO**（官方解释：New IO，也可称为Non Blocking IO）是从Java 1.4版本开始引入的一个新IO API，可以替代标准的Java IO API,NIO与原来的IO有同样的作用和目的，但是它们的最大区别是：IO是面向流的，NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。并且它们不能前后移动流中的数据，如果需要移动，需将它们缓存到一个缓冲区。而NIO的不同，它是将数据读取到一个等待处理的缓冲区，当需要移动时，可在缓冲区中进行移动，提高了灵活性、并可以进行更高效的读写操作。

​		NIO主要有三大核心部分：通道(Channel)、缓冲区(Buffer)、选择器(Selector)。通道表示打开的IO设备(如：文件、Socket等)的连接；缓冲区是一个用于特定基本类型的容器，由java.nio包定义，所有缓冲区都是Buffer抽象类的子类。若需要使用NIO，需要获取用于连接IO设备的通道以及用于容纳数据的缓冲区，然后操作缓冲区，对数据进行处理；选择器是用于监听多个通道的事件，可以同时监听多个IO状况，是非阻塞IO的核心。



**Channel(通道)**

Channel(通道) 用于源节点与目标节点的连接,在Java NIO中负责缓冲区中数据的传输，Channel本身不存储数据，因此需要配合缓冲区进行传输。Channel与IO中的Stream很像，但是Stream是单向的，而Channel是双向的，既可以进行读、又可以进行写。

主要实现类有：java.nio.channels.Channel接口

- FileChannel			             本地
- DatagramChannel             网络IO中UDP
- SocketChannel                   网络IO中TCP Client
- ServerSocketChannel        网络IO中TCP Server



**Buffer(缓冲区)**

Buffer(缓冲区)在java NIO中负责数据的存储。缓冲区就是数组，用于存储不同数据类型的数据。

根据数据类型不同（boolean除外），提供了相应类型的缓存区：

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

上述缓冲区的管理方式几乎一致，都是通过allocate()获取缓冲区



**Selector(选择器)**

Selector(选择器)可以让单个线程处理多个Channel，当你打开了多个管道，但是每个连接的流量都很低，那么使用Select就会很方便。它是SelectableChannle对象的多路复用器，Selector可以同时监听多个SelectableChannel的IO状况，也就是说，Selector可使一个单独的线程管理多个Channel。



### NIO主要区别

| IO                      | NIO                         |
| ----------------------- | --------------------------- |
| 面向流(Stream Oriented) | 面向缓冲区(Buffer Oriented) |
| 阻塞IO(Blocking IO)     | 非阻塞IO(Non Blocking IO)   |
| 无                      | 选择器(Selectors)           |



## 二、Buffer

### 2.1 Buffer的使用

- 分配空间ByteBuffer.allocate(1024)(此处使用ByteBuffer，其他类似)
- 写入数据buf.put("abcde".getBytes())
- 调用filp()方法切换成读取模式
- 从Buffer中读取数据buf.get()
- 调用clear()方法清空Buffer

上述已经大概介绍了Buffer的作用，现在进行比较下一步，Buffer中有几个非常重要的属性。

Buffer缓冲区中四个核心属性：

**capacity（容量）**：表示Buffer中最大存储数据的容量，缓冲区不能为负，并且一旦声明不能改变。

**limit（限制）**：表示缓冲区中可以操作数据的大小。(limit后数据不能进行读写)

**position（位置）**：表示Buffer下一个要读取或写入的操作数据的索引。

**mark（标记）**：表示记录当前position的位置。通过Buffer中的mark()方法指定Buffer中一个特定的position，可以通过reset()恢复到mark的位置。

Buffer源码：

````java
public abstract class Buffer {

    static final int SPLITERATOR_CHARACTERISTICS =
        Spliterator.SIZED | Spliterator.SUBSIZED | Spliterator.ORDERED;

    // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
    
    ....
}
````



![](http://picturebed.yifelix.cn/nio-1.jpg)

![](http://picturebed.yifelix.cn/nio-2.jpg)

注意：clear()方法只是将position设为0，limit设置为capactiy，,但是缓冲区中的数据依然存在，只是处于“被遗忘”状态，可以通过get继续获取其中数据。

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定的position，之后可以通过调用Buffer.reset()方法恢复到这个position。Buffer.rewind()方法将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素。

````java

        String str = "abcde";

        //1.分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);

        System.out.println("---------allocate---------");
        System.out.println(buf.position()); //0
        System.out.println(buf.limit());    //1024
        System.out.println(buf.capacity()); //1024

        //2.利用put()存入数据到缓存区中
        buf.put(str.getBytes());

        System.out.println("---------put()---------");
        System.out.println(buf.position());     //5
        System.out.println(buf.limit());        //1024
        System.out.println(buf.capacity());     //1024

        //3.切换成读取数据模式
        buf.flip();
        System.out.println("---------flip()---------");
        System.out.println(buf.position());     //0
        System.out.println(buf.limit());        //5
        System.out.println(buf.capacity());     //1024

        //4.利用get()读取缓冲区的数据
        byte[] dst = new byte[buf.limit()];
        buf.get(dst);
        System.out.println(new String(dst,0,dst.length));
        System.out.println("---------get()---------");
        System.out.println(buf.position());     //5
        System.out.println(buf.limit());        //5
        System.out.println(buf.capacity());     //1024

        //5.rewind() :可重复进行读取数据
        buf.rewind();
        System.out.println("---------rewind()---------");
        System.out.println(buf.position());     //0
        System.out.println(buf.limit());        //5
        System.out.println(buf.capacity());     //1024

        //6.clear()： 清空缓冲区,但是缓冲区中的数据依然存在，但是处于“被遗忘”状态
        buf.clear();
        System.out.println("---------clear()---------");
        System.out.println(buf.position());     //0
        System.out.println(buf.limit());        //1024
        System.out.println(buf.capacity());     //1024

        System.out.println((char) buf.get());   //a 数据仍然存在
````



### 2.2直接缓冲区和非直接缓冲区

- 非直接缓冲区：通过allocate() 方法分配缓冲区，将缓冲区建立在JVM的内存中
- 直接缓冲区: 通过allocateDirect() 方法分配直接缓冲区，将缓冲区建立在物理内存中,可以提高效率

![](http://picturebed.yifelix.cn/nio-3.jpg)

从上图可以看出非直接缓冲区创建的缓冲区，在JVM中内存中创建，在每次调用基础操作系统的一个本机IO之前或者之后，虚拟机都会将缓冲区的内容复制到中间缓冲区（或者从中间缓冲区复制内容），缓冲区的内容驻留在JVM内，因此销毁容易，但是占用JVM内存开销，处理过程中有复制操作。

​		而直接缓冲区创建的缓冲区，在JVM内存外开辟内存，在每次调用基础操作系统的一个本机IO之前或者之后，虚拟机都会避免将缓冲区的内容复制到中间缓冲区（或者从中间缓冲区复制内容），缓冲区的内容驻留在物理内存内，会少一次复制过程，如果需要循环使用缓冲区，用直接缓冲区可以很大地提高性能。虽然直接缓冲区使JVM可以进行高效的I/O操作，但它使用的内存是操作系统分配的，绕过了JVM堆栈，建立和销毁比堆栈上的缓冲区要更大的开销。所以建议将直接缓冲区主要分配给那些易受基础系统的本机I/O操作影响的大型、持久的缓冲区。

````java
//分配直接缓冲区
ByteBuffer buf = ByteBuffer.allocateDirect(1024);
//判断是否是直接缓冲区
System.out.println(buf.isDirect());
````

## 三、Channel

![](http://picturebed.yifelix.cn/nio-4.jpg)

通道：由java.nio.channels包定义。
Channel表示IO源与目标打开的连接。
Channel类似于传统的“流”。但其自身不能直接访问数据，Channel只能与Buffer进行交互。

**主要获取通道的方式：**

- Java针对支持通道的类提供了getChannel()方法

  - 本地IO：

    FileInputStream/FileOutputStream

    RandomAccessFile

  - 网络IO：

    Socket

    ServerSocket

    DatagramSocket

- 在JDK1.7中的NIO.2针对各个通道提供了静态方法open()
- 在JDK1.7中的NIO.2的Files工具类的newByteChannel()



**利用通道完成文件的复制(非直接缓冲区)**

````java
FileInputStream fis = null;
FileOutputStream fos = null;
FileChannel inChannel = null;
FileChannel outChannel = null;
try {
    //使用本地IO流
    fis = new FileInputStream("1.png");
    fos = new FileOutputStream("2.png");

    //①获取通道
    inChannel = fis.getChannel();
    outChannel = fos.getChannel();

    //②分配指定代销的缓冲区
    ByteBuffer buf = ByteBuffer.allocate(1024);

    //③将通道中的数据存入缓冲区中
    while (inChannel.read(buf) != -1) {
        buf.flip(); //切换读取数据的模式
        //④将缓冲区中的数据写入通道
        outChannel.write(buf);
        buf.clear();    //清空缓冲区
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    //关闭通道
    if (fos != null) {
        try {
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    if (outChannel != null) {
        try {
            outChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    if (inChannel != null) {
        try {
            inChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    if (fis != null) {
        try {
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
````



**使用直接缓冲区完成文件的复制(内存映射文件)**

````hava
FileChannel inChannel = null;
FileChannel OutChannel = null;
try {
	//通过静态方法open()创建通道
    inChannel = FileChannel.open(Paths.get("1.png"), StandardOpenOption.READ);
    OutChannel = FileChannel.open(Paths.get("3.png"), StandardOpenOption.WRITE,
            StandardOpenOption.READ,StandardOpenOption.CREATE);
    //CREATE存在就覆盖 CREATE_NEW存在就覆盖

    //内存映射文件
    MappedByteBuffer inMappedBuf = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
    MappedByteBuffer outMappedBuf = OutChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());

    //直接对缓冲区进行数据的读写操作
    byte[] dst = new byte[inMappedBuf.limit()];
    inMappedBuf.get(dst);
    outMappedBuf.put(dst);

} catch (IOException e) {
    e.printStackTrace();
}finally{
	//关闭通道
    if(inChannel != null){
        try {
            inChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    if(OutChannel != null){
        try {
            OutChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
````

**在上面两份代码的进行运行效果测试时，使用直接缓冲区会比非直接缓冲区速度快上很多，在运行的过程中缓冲区的内容驻留在物理内存内，会少一次复制过程，但直接缓冲区有个弊端，在运行完毕之后因为是在物理内存中，它不是由JVM所管理，所以不能立即就进行垃圾回收，可能会占用内存空间。**



### 分散(Scatter)与聚集(Gather)

- **分散读取(Scattering Rads)：将通道中的数据分散到多个缓冲区中**（注意：按照缓冲区的顺序，从Channel中读取的数据依次将Buffer填满）

![](http://picturebed.yifelix.cn/nio-5.jpg)

- **聚集写入(Gathering Writes)：将多个缓冲区中的数据聚集到通道中**（注意：按照缓冲区的顺序，写入position和limit之间的数据到Channel）

![](http://picturebed.yifelix.cn/nio-6.jpg)

````java
RandomAccessFile raf1 = null;
try {
    //使用本地IO
    raf1 = new RandomAccessFile("read.txt", "rw");

    //获取通道
    FileChannel channel1 = raf1.getChannel();

    //分配指定大小的缓冲区
    ByteBuffer buf1 = ByteBuffer.allocate(100);
    ByteBuffer buf2 = ByteBuffer.allocate(1024);

    //3.分散读取
    ByteBuffer[] bufs = {buf1, buf2};
    channel1.read(bufs);

    //将全部缓冲区切换为读模式
    for (ByteBuffer byteBuffer : bufs) {
        byteBuffer.flip();
    }
    //遍历缓冲区中的数据
    for (int i = 0; i < bufs.length; i++) {
        System.out.println(new String(bufs[i].array(), 0, bufs[i].limit()));
    }

    //聚集写入
    RandomAccessFile raf2 = new RandomAccessFile("write.txt","rw");
    FileChannel channel2 = raf2.getChannel();
    channel2.write(bufs);

} catch (IOException e) {
    e.printStackTrace();
} finally {
    //关闭通道
    if (raf1 != null) {
        try {
            raf1.close();
        } catch (IOException e) {
            
            e.printStackTrace();
        }
    }
}
````



### 字符集

JDK1.4提供了Charset来处理字节序列和字符序列之间的转换关系,可对文本文件进行编码：明文的字符序列转化成计算机里的二进制序列（encoder），解码：二进制序列转化为明文字符序列（decoder）。

**可查看CharSet支持的字符集:**

````java
SortedMap<String, Charset> map = Charset.availableCharsets();
Set<Map.Entry<String, Charset>> set = map.entrySet();

for (Map.Entry<String, Charset> entry : set) {
    System.out.println(entry.getKey() + "=" + entry.getValue());
}
````

**对相应字符集进行编码和解码：**

````java
//获取对应字符集的CharSet
Charset cs1 = Charset.forName("GBK");
//获取编码器
CharsetEncoder ce = cs1.newEncoder();
//获取解码器
CharsetDecoder cd = cs1.newDecoder();

CharBuffer cBuf = CharBuffer.allocate(1024);    //创建缓冲区
cBuf.put("测试"); //写入数据
cBuf.flip();      //转换成读取模式

//编码  将字符串转换成字节
ByteBuffer bBuf = ce.encode(cBuf);
//GBK格式下中文一个占用两个位置
for (int i = 0; i < cBuf.limit() * 2; i++) {
    System.out.println(bBuf.get());
}

bBuf.flip();
//解码
CharBuffer cBuf2 = cd.decode(bBuf);
System.out.println(cBuf2.toString());
````

## 四、Selector

Selector(选择器)是 SelectableChannle 对象的多路复用器，Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector可使一个单独的线程管理多个 **Channel**。Selector 是非阻塞 IO 的核心。

而阻塞与非阻塞的概念中，传统IO流都是阻塞式的。也就是说，当一个线程调用read()或write()时，该线程是被阻塞的，只能进行数据被读取或是写入，这个线程不能进行其他任务。而NIO是非阻塞模式，非阻塞IO是当线程从某通道进行读写时，若没有数据可用时，该线程可以进行其他任务。举个例子：在阻塞模式时，小明去烧水，然后在那等待水烧开，此时什么事情都不能干；而非阻塞模式时，小明去烧水，在烧水期间小明去客厅看了个电视干了点别的事情，这就是阻塞模式与非阻塞模式的区别。

**非阻塞IO简单应用：**

````java
/**
* TCP通信
*/

//客户端
public void client() throws IOException {
    //1.获取通道
    SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 10086));

    //2.切换非阻塞模式
    sChannel.configureBlocking(false);

    //3.分配指定大小的缓冲区
    ByteBuffer buf = ByteBuffer.allocate(1024);

    //4.发送数据给服务端
    buf.put(new Date().toString().getBytes());
    buf.flip();
    sChannel.write(buf);
    buf.clear();

    //5.关闭通道
    sChannel.close();

}

//服务端
public void server() throws IOException {
    //1.获取通道
    ServerSocketChannel ssChannel=ServerSocketChannel.open();

    //2.切换非阻塞式模式
    ssChannel.configureBlocking(false);

    //3.绑定连接
    ssChannel.bind(new InetSocketAddress(10086));

    //4.获取选择器
    Selector selector=Selector.open();

    //5.将通道注册到选择器上，并且指定“监听接收事件”
    ssChannel.register(selector,SelectionKey.OP_ACCEPT);

    //6.轮询式的获取选择器上已经“准备就绪”的事件
    while(selector.select()>0){

        //7.获取当前选择器中所有注册的“选择键（已就绪的监听事件）”
        Iterator<SelectionKey> it=selector.selectedKeys().iterator();

        while(it.hasNext()){
            //8.获取准备“就绪”的事件
            SelectionKey sk=it.next();

            //9.判断具体是什么时间准备就绪
            if(sk.isAcceptable()){
                //10.若“接收就绪”，获取客户端连接
                SocketChannel sChannel=ssChannel.accept();

                //11.切换非阻塞模式
                sChannel.configureBlocking(false);

                //12.将该通道注册到选择器上
                sChannel.register(selector, SelectionKey.OP_READ);
            }else if(sk.isReadable()){
                //13.获取当前选择器上“读就绪”状态的通道
                SocketChannel sChannel=(SocketChannel)sk.channel();
                //14.读取数据
                ByteBuffer buf=ByteBuffer.allocate(1024);
                int len=0;
                while((len=sChannel.read(buf))>0){
                    buf.flip();
                    System.out.println(new String(buf.array(),0,len));
                    buf.clear();
                }
            }
            //15.取消选择键SelectionKey
            it.remove();
        }
    }
}
````

````java
/**
* UDP通信
*/
 public void send() throws IOException {
    DatagramChannel dc=DatagramChannel.open();
    dc.configureBlocking(false);
    ByteBuffer buf=ByteBuffer.allocate(1024);
    Scanner scan=new Scanner(System.in);
    while(scan.hasNext()){
        String str=scan.next();
        buf.put((new Date().toString()+"\n"+str).getBytes());
        buf.flip();
        dc.send(buf, new InetSocketAddress("127.0.0.1", 10086));
        buf.clear();
    }
    dc.close();
}

public void receive() throws IOException {
    DatagramChannel dc = DatagramChannel.open();

    dc.configureBlocking(false);

    dc.bind(new InetSocketAddress(10086));

    Selector selector = Selector.open();

    dc.register(selector, SelectionKey.OP_READ);

    while(selector.select() > 0){
        Iterator<SelectionKey> it = selector.selectedKeys().iterator();

        while(it.hasNext()){
            SelectionKey sk = it.next();

            if(sk.isReadable()){
                ByteBuffer buf = ByteBuffer.allocate(1024);

                dc.receive(buf);
                buf.flip();
                System.out.println(new String(buf.array(),0,buf.limit()));
                buf.clear();
            }
        }

        it.remove();
    }
}
````



在服务端进行通道注册调用register(Selector sel, int ops)进行将通道注册到选择器中，只有当通道中完成一些事件时，线程才会进行完成相应的方法。

可以监听的事件类型（可使用SelectionKey的四个常量表示）：

- 读：SelectionKey.OP_READ  （1）
- 写：SelectionKey.OP_WRITE  （4）
- 连接：SelectionKey.OP_CONNECT  （8）
- 接收：SelectionKey.OP_ACCEPT  （16）

若注册时不止监听一个事件，则可以使用”位或“操作符连接。

