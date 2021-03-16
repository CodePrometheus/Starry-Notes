# I/O

[TOC]





## 概念

Java的IO流是实现输入/输出的基础，它可以方便地实现数据的输入/输出操作，在Java中把不同的输入/输出源抽象表述为"流"，Java对数据的操作是通过流的方式，用于操作流的对象都在IO包中

流  ->  流是一组有顺序的有起点和终点的字节集合，是对数据传输的总称或抽象，即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作，流有输入和输出，输入时是流从数据源流向程序，输出时是流从程序传向数据源，而数据源可以是内存，文件，网络或程序等

> 分类：
>
> 1. 按照数据流向
>
>    - 输入流  只读入数据
>    - 输出流   只写入数据
>
> 2. 按照数据单位  （以下四个流对象都是抽象类，四种抽象基类，不能直接使用，需要使用子类）
>
>    - 字节流（8 bit）
>      - 字节输入流 InputStream 读
>      - 字节输出流 OutputStream 写
>
>    - 字符流 （16 bit）=字节流+编码表  只能读取字符类型数据
>      - 字符输入流 Reader 读
>      - 字符输出流 Writer 写
>
> 3. 按照流的角色
>
>    - 节点流   (直接与数据源相连)
>      - 可以从/向一个特定的IO设备（如磁盘、网络）读/写数据的流，也称低级流
>    - 处理流    
>      - 对一个已存在的流进行连接或封装，通过封装后的流来实现数据读/写功能，也称高级流



## File类

File类是java.io包下代表与平台无关的文件和目录，也就是说，如果希望在程序中操作**文件和目录**，都可以通过File类来完成，file类的一个对象，代表一个文件或一个文件目录

```java
//构造器
File(File parent,String child)
File(String pathname)
File(String parent,String child)
File(URI uri)
```

- 相对路径：
  - ./表示当前路径
  - ../表示上一级路径
  - 其中当前路径：默认情况下，java.io 包中的类总是根据当前用户目录来分析相对路径名。此目录由系统属性 user.dir 指定，通常是 Java 虚拟机的调用目录
  - 注意Windows和DOS默认使用"\\"来表示，Linux\unix\URL使用"/"表示
  - 也可以直接使用代码`File.separator`，表示跨平台分隔符
- 绝对路径：
  - 绝对路径名是完整的路径名，不需要任何其他信息就可以定位自身表示的文件





### File获取

```java
String getName();//返回文件或者是目录的名称
String getPath();//返回路径
String getAbsolutePath();//返回绝对路径
String getParent();//返回父目录，如果没有父目录则返回null
long lastModified();//返回最后一次修改的时间
long length();//返回文件的长度
File[] listRoots();// 列出所有的根目录（Window中就是所有系统的盘符）
String[] list() ;//返回一个字符串数组，给定路径下的文件或目录名称字符串
String[] list(FilenameFilter filter);//返回满足过滤器要求的一个字符串数组
File[]  listFiles();//返回一个文件对象数组，给定路径下文件或目录
File[] listFiles(FilenameFilter filter);//返回满足过滤器要求的一个文件对象数组
```



### File判断

```java
boolean canExecute()    ;//判断文件是否可执行
boolean canRead();//判断文件是否可读
boolean canWrite();//判断文件是否可写
boolean exists();//判断文件是否存在
boolean isDirectory();//判断是否是目录
boolean isFile();//判断是否是文件
boolean isHidden();//判断是否是隐藏文件或隐藏目录
boolean isAbsolute();//判断是否是绝对路径 文件不存在也能判断
```



### File创建

```java
//如果文件存在返回false，否则返回true并且创建文件 
boolean createNewFile();
//创建一个File对象所对应的目录，成功返回true，否则false。且File对象必须为路径而不是文件。只会创建最后一级目录，如果上级目录不存在就抛异常。
boolean mkdir();
//创建一个File对象所对应的目录，成功返回true，否则false。且File对象必须为路径而不是文件。创建多级目录，创建路径中所有不存在的目录
boolean mkdirs()    ;
//如果文件存在返回true并且删除文件，否则返回false
boolean delete();
//在虚拟机终止时，删除File对象所表示的文件或目录。
void deleteOnExit();
```





## 输入

InputStream 是所有的输入字节流的父类，它是一个抽象类，主要包含三个方法

```java
//读取一个字节并以整数的形式返回(0~255),如果返回-1已到输入流的末尾。 
int read() ； 
//读取一系列字节并存储到一个数组buffer，返回实际读取的字节数，如果读取前已到输入流的末尾返回-1。 
int read(byte[] buffer) ； 
//读取length个字节并存储到一个字节数组buffer，从off位置开始存,最多len， 返回实际读取的字节数，如果读取前以到输入流的末尾返回-1。 
int read(byte[] buffer, int off, int len) ；
```



Reader 是所有的输入字符流的父类，它是一个抽象类，主要包含三个方法

```java
//读取一个字符并以整数的形式返回(0~255),如果返回-1已到输入流的末尾。 
int read() ； 
//读取一系列字符并存储到一个数组buffer，返回实际读取的字符数，如果读取前已到输入流的末尾返回-1。 
int read(char[] cbuf) ； 
//读取length个字符,并存储到一个数组buffer，从off位置开始存,最多读取len，返回实际读取的字符数，如果读取前以到输入流的末尾返回-1。 
int read(char[] cbuf, int off, int len)
```

- 对比InputStream和Reader所提供的方法，就不难发现两个基类的功能基本一样的，只不过读取的数据单元不同。
  - **在执行完流操作后，要调用**`close()`**方法来关系输入流，因为程序里打开的IO资源不属于内存资源，垃圾回收机制无法回收该资源，所以应该显式关闭文件IO资源。**
- 除此之外，InputStream和Reader还支持如下方法来移动流中的指针位置

```java
//在此输入流中标记当前的位置
//readlimit - 在标记位置失效前可以读取字节的最大限制。
void mark(int readlimit)
// 测试此输入流是否支持 mark 方法
boolean markSupported()
// 跳过和丢弃此输入流中数据的 n 个字节/字符
long skip(long n)
//将此流重新定位到最后一次对此输入流调用 mark 方法时的位置
void reset()
```



## 输出

OutputStream 是所有的输出字节流的父类，它是一个抽象类，主要包含如下四个方法

```java
//向输出流中写入一个字节数据,该字节数据为参数b的低8位。 
void write(int b) ; 
//将一个字节类型的数组中的数据写入输出流。 
void write(byte[] b); 
//将一个字节类型的数组中的从指定位置（off）开始的,len个字节写入到输出流。 
void write(byte[] b, int off, int len); 
//将输出流中缓冲的数据全部写出到目的地。 
void flush();
```



Writer 是所有的输出字符流的父类，它是一个抽象类,主要包含如下六个方法

```java
//向输出流中写入一个字符数据,该字节数据为参数b的低16位。 
void write(int c); 
//将一个字符类型的数组中的数据写入输出流， 
void write(char[] cbuf) 
//将一个字符类型的数组中的从指定位置（offset）开始的,length个字符写入到输出流。 
void write(char[] cbuf, int offset, int length); 
//将一个字符串中的字符写入到输出流。 
void write(String string); 
//将一个字符串从offset开始的length个字符写入到输出流。 
void write(String string, int offset, int length); 
//将输出流中缓冲的数据全部写出到目的地。 
void flush()
```

Writer比OutputStream多出两个方法，主要是支持写入字符和字符串类型的数据。





## 缓冲流

缓冲流是对文件流处理的一种流，是处理流的一种，它本身并不具备 IO 功能，只是在别的流上加上缓冲提高了效率，当对文件或其他目标频繁读写或操作效率低，效能差，这时使用缓冲流能够更高效的读写信息。因为缓冲流先将数据缓存起来，然后一起写入或读取出来。所以说，缓冲流还是很重要的，在IO操作时记得加上缓冲流提升性能。

BufferedInputStream 和BufferedOutputStream 这两个类分别是InputStream 和OutputStream 的子类，作为装饰器子类，使用它们可以防止每次读取/发送数据时进行实际的写操作，代表着使用缓冲区。

不带缓冲的操作，每读一个字节就要写入一个字节，由于涉及磁盘的IO操作相比内存的操作要慢很多，所以不带缓冲的流效率很低；带缓冲的流，可以一次读很多字节，但不向磁盘中写入，只是先放到内存里，等凑够了缓冲区大小的时候一次性写入磁盘，这种方式可以减少磁盘操作次数，速度就会提高很多

同时正因为它们实现了缓冲功能，所以要注意在使用BufferedOutputStream写完数据后，要调用flush()方法或close()方法，强行将缓冲区中的数据写出。否则可能无法写出数据。与之相似还BufferedReader 和BufferedWriter 两个类

> flush 是从缓冲区把文件写出，
>
> close 是将文件从缓冲区内写出并且关闭相应的流。
>
> 缓冲流的数据存储在缓冲区，所以必须强行的将数据从缓冲区写出，否则可能无法写出是数据

```java
//先关闭外层的流，在关闭内层的流
bos.close();
bis.close();
//关闭外层流的同时，内层流也会自动的进行关闭，故内层流的关闭可以省略
fos.close();
fis.close();
```



## 转换流

提供了在字节流和字符流之间的转换

|                      InputStreamReader                       |                      OutputStreamWriter                      |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| 将InputStream转换为Reader，一个字节的输入流转换为字符的输入流 | 将Writer转换为OutputStream，一个字符的输出流转换为字节的输出流 |

字节流中的数据都是字符时，转成字符流操作更高效

常用转换流来处理文件乱码问题，实现编码和解码的功能

1. 解码：字节、字节数组   —>  字符数组、字符串
2. 编码：字符数组、字符串  —>  字节、字节数组





## IO体系

|      分类      |      字节输入流      |      字节输出流       |    字符输入流     |     字符输出流     |
| :------------: | :------------------: | :-------------------: | :---------------: | :----------------: |
|  **抽象基类**  |     InputStream      |     OutputStream      |      Reader       |       Writer       |
|  **访问文件**  |   FileInputStream    |   FileOutputStream    |    FileReader     |     FileWriter     |
|  **访问数组**  | ByteArrayInputStream | ByteArrayOutputStream |  CharArrayReader  |  CharArrayWriter   |
|  **访问管道**  |   PipedInputStream   |   PipedOutputStream   |    PipedReader    |    PipedWriter     |
| **访问字符串** |                      |                       |   StringReader    |    StringWriter    |
|   **缓冲流**   | BufferedInputStream  | BufferedOutputStream  |  BufferedReader   |   BufferedWriter   |
|   **转换流**   |                      |                       | InputStreamReader | OutputStreamReader |
|   **打印流**   |                      |      PrintStream      |                   |    PrintWriter     |
| **推回输入流** | PushbackInputStream  |                       |  PushbackReader   |                    |
|   **特殊流**   |   DataInputStream    |   DataOutputStream    |                   |                    |
|   **对象流**   |  ObjectInputStream   |  ObjectOutputStream   |                   |                    |
|                |    **字节输入流**    |    **字节输出流**     |  **字符输入流**   |   **字符输出流**   |

