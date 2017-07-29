---
title: Java IO
date: 2017-06-27 19:08:03
tags: [java, IO, NIO, NIO2]
categories: java
---

___主要内容___

I/O 就是数据的输入/输出。 Java 平台提供了丰富的类库来满足可能的I/O操作需求。
最初的java.io包， JDK 1.4 的 NIO， JDK 7 的 NIO.2。
最早的java.io包把IO操作抽象成数据的流动，进而有了流(Stream)概念；
Java NIO中把IO操作抽象成端到端的一个数据连接，进而有了通道(channel)概念。
如果需要开发高性能网络应用， Java提供的标准库所支持的抽象层次过低， 推荐Netty。

JDK 7 后都建议用 try-with-resources来使用流和通道。

<!-- more -->

## 流
java中 最基本的流就是在字节这个层次上处理的。

### 基本输入流 `InputStream`
#### 数据功能
`read`方法
+ 每次读一个字节
+ 字节数组缓冲区 
最常用， 循环读取， 直到返回-1或异常
+ 字节数组缓冲区和起始位置和长度

#### 控制功能
+ close 关闭
+ skip 跳过指定数目的字节
+ mark 和 reset 标记和重置配合，实现流中部分内容的重复读取
不是所有流都支持mark， 需要用 `markSupperted()`检查
+ available 非阻塞的获取可供读取的字节数


### 基本输出流 `OutputStream`
#### 数据功能
`write` 方法， 类似 `read` 一样， 3种重载形式：每次写入一个字节, 也可写入一个字节数组的全部或部分内容。

#### 控制功能
+ close 关闭
+ flush 强制保存在缓冲区的内容立即进行实际的写入操作。

### 输入流复用
两种方式：
+ 利用标记和重置
使用`BufferedInputStream`
+ 输入流转换成数据
直接把流中所有数据读取到一个字节数组中

这两种复用流的思路一致， 就是预先把需要复用的数据保存起来。

### 过滤输入输出流
+ 缓冲区
BufferedInputStream， BufferedOutputStream
+ java基本类型支持
DataInputStream， DataOutputStream
+ java 基本类型和对象支持
ObjectInputStream， ObjectOutputStream
+ 数据回流
PushbackInputStream， PushbackOutputStream
+ 文件流
FileInputStream， FileOutputStream
+ 管道流
PipedInputStream， PipedOutputStream
+ 顺序连接
SequenceInputStream， SequenceOutputStream

### 字符流
  以上所有都是字节流， 字节流更适合机器处理。对于人来说，我们更愿意看到直接可读的字符。在在 java.io 包中还有一组类用来处理字符流,即 java.io.Reader 类和 java.io.Writer 
类及其子类。这些字符流处理的是字符类型,而不是字节类 型。字符流适合用于处理程序中包含的文本类型的内容。
  创建一个字符流的最常见做法是通过一个字节流 InputStream 类或 OutputStream 类 的对象进行创建,对应的是 InputStreamReader 类和 OutputStreamWriter 类。在从字节流转换成字符流时,需要指定字符的编码格式。如果编码格式错误,会产生包含乱码的字符串。

+ 字符
java.io.StringReader 类 和 java.io.StringWriter 
+ 字符数组
 java. io.CharArrayReader 类和 java.io.CharArrayWriter 类
+ 内部缓冲区
java.io.BufferedReader 类 和 java.io.BufferedWriter 类

## 缓冲区
  Java NIO 中的缓冲区在某些特性上类似于 Java 中的基本类型的数组(如字节数组 或整型数组等),比如缓冲区中的数据排列是线性的,缓冲区的空间也是有限的。
不过缓冲区所提供的功能远比数组丰富得多,而且也支持存储类型异构的数据。

### 基本概念
容量(capacity)、读写限制(limit)和读写位置(position)

+ 容量
容量指的是缓冲区的额定大小。容量是在创建缓冲区时指定的,无法在创建后更改。在任何时候缓冲区中的数据总数都不可能超过容量。
只读

+ 读写限制
在缓冲区中进行读写操作时的最大允许位置。
可读读写

+ 读写位置
表示的是当前进行读写操作时的位置。
可读可写

缓冲区同样也支持标记 mark 和重置 reset 的特性。

任何时候 `0 <= 标记位置 <= 读写位置 <= 读写限制 <= 容量`  656404562

java.nio.Buffer 类是所有不同数据类型的缓冲区的父类。
+ clear方法
把读写限制设为缓冲区的容量,同时 把读写位置设为 0 

+ flip
读写限制设为当前的读写位置,再把读写位置设为 0,这样可以保证缓冲区中的全部 数据都可以被读取

+ rewind
不会改变读写限制,但是会把读写位置设为 0


读写操作分成两类 :一类是根据当前读写位置进行的相对读写操作, 另外一类是根据在缓冲区中的绝对位置进行的读写操作。两者的差别在于相对读写会改变当前读写位置,而绝对读写则不会。

### 字节缓冲区
java.nio.ByteBuffer 类
只能通过其静态工厂方法 allocate 来分配新空间, 或者通过 wrap 方法来包装一个已有的字节数组。
必须要考虑字节顺序，同样的字节序列按照不同的顺序去解释,所得到的结果是不同的。
java.nio.ByteOrder 类中定义了两种最基本的字节顺序 :BIG_ENDIAN 对应的大端表示和 LITTLE_ENDIAN 对应的小端表示。
大端表示的含义是字节序列中高位在前,而小端表示则正好相反。ByteOrder类中的静态方法 nativeOrder 可以获取到底层操作系统平台采用的字节顺序。ByteBuffer 类的对象默认使用的是大端表示。

### 缓冲区视图
ByteBuffer 类的另外一个常见的使用方式是在一个已有的 ByteBuffer 类的对象上创建出各种不同的视图。这些视图和它所基于的 ByteBuffer 类的对象共享同样的存储空间,但是提供额外的实用功能。在功能上,ByteBuffer 类的视图与它所基于的 ByteBuffer 类的对象之间的关系类似于过滤流和它所包装的流的关系。
CharBuffer、ShortBuffer、IntBuffer、LongBuffer、FloatBuffer 和 DoubleBuffer 。


## 通道
通道是 Java NIO 对 I/O 操作提供的另外一种新的抽象方式。通道不是从 I/O 操作所处理的数据这个层次上去抽象,而是表示为一个已经建立好的到支持 I/O 操作的实体的连接。这个连接一旦建立,就可以在这个连接上进行各种 I/O 操作。通道在进行读写操作 时使用的都是上面介绍的新的缓冲区的实现,而不是字节数组。

Java NIO 中 的 通 道 都 实 现 了 java.nio.channels.Channel 接 口， 本身很简单，只有关闭通道的 close 方法和判断通道是否被打开的 isOpen 方法。
由于 Channel 接口继承了 java.lang.AutoCloseable 接口,通道的所有实现对象都可以用 try-with-resources 。

+ 读 java.nio.channels.ReadableByte-Channel 接口 
+ 写 java.nio.channels. WritableByteChannel 接口
+ 移动读写操作的位置 java.nio.channels.SeekableByteChannel 接口
+ 多个 ByteBuffer 类数据 java.nio.channels.ScatteringByteChannel

### 文件通道FileChannel

#### 基本
读取和写入模式 java.nio.file.StandardOpenOption

打开方法：
+ FileChannel.open(Path path)
+ 从已有的 FileInputStream 类、FileOutputStream 类和 RandomAccessFile 类的对象的getChannel 方法得到

#### 文件数据传输
transferFrom
transferTo

#### 内存映射文件
+ map 方法
FileChannel 类的 map 方法把一个文件的全部或部分内容映射到内存中,所 得到的是一个 ByteBuffer 类的子类 MappedByteBuffer 的对象,程序只需要对这个 MappedByteBuffer 类的对象进行操作即可。
+ load 方法
MappedByteBuffer 类的 load 方法可以把该缓冲区所对应的文件内容 加载到物理内存中,以提高文件操作时的性能。
+ force 方法
FileChannel 类的 force 方法来强制要求把这些更新同 步到底层文件中。

#### 锁定文件
FileChannel 类的 lock 和 tryLock 方 法可以对当前文件通道所对应的文件进行加锁。
lock 方法是阻塞式的,而 tryLock 方法则不是

### 套接字通道
接口 java.nio.channels.NetworkChannel 

#### 阻塞式套接字通道
客户端 SocketChannel.open
服务端 ServerSocketChannel.open

#### 多路复用套接字通道
核心是选择器,即 java.nio.channels.Selector 类


## NIO.2
### 文件系统
java.nio.file包， 提供的新功能包括文件路径的抽象、文件目录列表流、文件目录树遍历、文件 属性和文件变化监视服务等。
传统的是 `java.io.File` 类

#### 文件路径
java.nio.file.Path 接口

#### 文件目录列表流
接口 java.nio.file.DirectoryStream
只能遍历当前目录下的直接子目录或文件， 不递归。

#### 文件目录树遍历
java.nio.file.FileVisitor 接口

#### 文件属性
java.nio.file.attribute 包

#### 文件操作
java.nio.file.Files 类
[jdk docs](https://docs.oracle.com/javase/7/docs/api/java/nio/file/Files.html)

```java

static long	copy(InputStream in, Path target, CopyOption... options)
Copies all bytes from an input stream to a file.
static long	copy(Path source, OutputStream out)
Copies all bytes from a file to an output stream.
static Path	copy(Path source, Path target, CopyOption... options)
Copy a file to a target file.
static Path	createDirectories(Path dir, FileAttribute<?>... attrs)
Creates a directory by creating all nonexistent parent directories first.
static Path	createDirectory(Path dir, FileAttribute<?>... attrs)
Creates a new directory.
static Path	createFile(Path path, FileAttribute<?>... attrs)
Creates a new and empty file, failing if the file already exists.
static Path	createLink(Path link, Path existing)
Creates a new link (directory entry) for an existing file (optional operation).
static Path	createSymbolicLink(Path link, Path target, FileAttribute<?>... attrs)
Creates a symbolic link to a target (optional operation).
static Path	createTempDirectory(Path dir, String prefix, FileAttribute<?>... attrs)
Creates a new directory in the specified directory, using the given prefix to generate its name.
static Path	createTempDirectory(String prefix, FileAttribute<?>... attrs)
Creates a new directory in the default temporary-file directory, using the given prefix to generate its name.
static Path	createTempFile(Path dir, String prefix, String suffix, FileAttribute<?>... attrs)
Creates a new empty file in the specified directory, using the given prefix and suffix strings to generate its name.
static Path	createTempFile(String prefix, String suffix, FileAttribute<?>... attrs)
Creates an empty file in the default temporary-file directory, using the given prefix and suffix to generate its name.
static void	delete(Path path)
Deletes a file.
static boolean	deleteIfExists(Path path)
Deletes a file if it exists.
static boolean	exists(Path path, LinkOption... options)
Tests whether a file exists.
static Object	getAttribute(Path path, String attribute, LinkOption... options)
Reads the value of a file attribute.
static <V extends FileAttributeView> 
V	getFileAttributeView(Path path, Class<V> type, LinkOption... options)
Returns a file attribute view of a given type.
static FileStore	getFileStore(Path path)
Returns the FileStore representing the file store where a file is located.
static FileTime	getLastModifiedTime(Path path, LinkOption... options)
Returns a file's last modified time.
static UserPrincipal	getOwner(Path path, LinkOption... options)
Returns the owner of a file.
static Set<PosixFilePermission>	getPosixFilePermissions(Path path, LinkOption... options)
Returns a file's POSIX file permissions.
static boolean	isDirectory(Path path, LinkOption... options)
Tests whether a file is a directory.
static boolean	isExecutable(Path path)
Tests whether a file is executable.
static boolean	isHidden(Path path)
Tells whether or not a file is considered hidden.
static boolean	isReadable(Path path)
Tests whether a file is readable.
static boolean	isRegularFile(Path path, LinkOption... options)
Tests whether a file is a regular file with opaque content.
static boolean	isSameFile(Path path, Path path2)
Tests if two paths locate the same file.
static boolean	isSymbolicLink(Path path)
Tests whether a file is a symbolic link.
static boolean	isWritable(Path path)
Tests whether a file is writable.
static Path	move(Path source, Path target, CopyOption... options)
Move or rename a file to a target file.
static BufferedReader	newBufferedReader(Path path, Charset cs)
Opens a file for reading, returning a BufferedReader that may be used to read text from the file in an efficient manner.
static BufferedWriter	newBufferedWriter(Path path, Charset cs, OpenOption... options)
Opens or creates a file for writing, returning a BufferedWriter that may be used to write text to the file in an efficient manner.
static SeekableByteChannel	newByteChannel(Path path, OpenOption... options)
Opens or creates a file, returning a seekable byte channel to access the file.
static SeekableByteChannel	newByteChannel(Path path, Set<? extends OpenOption> options, FileAttribute<?>... attrs)
Opens or creates a file, returning a seekable byte channel to access the file.
static DirectoryStream<Path>	newDirectoryStream(Path dir)
Opens a directory, returning a DirectoryStream to iterate over all entries in the directory.
static DirectoryStream<Path>	newDirectoryStream(Path dir, DirectoryStream.Filter<? super Path> filter)
Opens a directory, returning a DirectoryStream to iterate over the entries in the directory.
static DirectoryStream<Path>	newDirectoryStream(Path dir, String glob)
Opens a directory, returning a DirectoryStream to iterate over the entries in the directory.
static InputStream	newInputStream(Path path, OpenOption... options)
Opens a file, returning an input stream to read from the file.
static OutputStream	newOutputStream(Path path, OpenOption... options)
Opens or creates a file, returning an output stream that may be used to write bytes to the file.
static boolean	notExists(Path path, LinkOption... options)
Tests whether the file located by this path does not exist.
static String	probeContentType(Path path)
Probes the content type of a file.
static byte[]	readAllBytes(Path path)
Reads all the bytes from a file.
static List<String>	readAllLines(Path path, Charset cs)
Read all lines from a file.
static <A extends BasicFileAttributes> 
A	readAttributes(Path path, Class<A> type, LinkOption... options)
Reads a file's attributes as a bulk operation.
static Map<String,Object>	readAttributes(Path path, String attributes, LinkOption... options)
Reads a set of file attributes as a bulk operation.
static Path	readSymbolicLink(Path link)
Reads the target of a symbolic link (optional operation).
static Path	setAttribute(Path path, String attribute, Object value, LinkOption... options)
Sets the value of a file attribute.
static Path	setLastModifiedTime(Path path, FileTime time)
Updates a file's last modified time attribute.
static Path	setOwner(Path path, UserPrincipal owner)
Updates the file owner.
static Path	setPosixFilePermissions(Path path, Set<PosixFilePermission> perms)
Sets a file's POSIX permissions.
static long	size(Path path)
Returns the size of a file (in bytes).
static Path	walkFileTree(Path start, FileVisitor<? super Path> visitor)
Walks a file tree.
static Path	walkFileTree(Path start, Set<FileVisitOption> options, int maxDepth, FileVisitor<? super Path> visitor)
Walks a file tree.
static Path	write(Path path, byte[] bytes, OpenOption... options)
Writes bytes to a file.
static Path	write(Path path, Iterable<? extends CharSequence> lines, Charset cs, OpenOption... options)
Write lines of text to a file.
```



## 异步 I/O 通道
java.nio.channels.AsynchronousFileChannel 
通过 open 方法来完成的。 对文件通道的读取和写入也是通过对应的 read 和 write 方法来完成的。所不同的是 read 和 write 方法要么返回一个 Future 类的对象,要么要求传入一个 CompletionHandler 接口 的实现对象作为回调方法。

异 步 套 接 字 通 道 AsynchronousSocketChannel 和 AsynchronousServerSocketChannel 类 分 别 对 应 一 般 的 SocketChannel 和 ServerSocketChannel 类。
 
