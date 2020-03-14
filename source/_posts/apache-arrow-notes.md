---
title: apache_arrow_notes
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2020-02-27 13:07:51
tags:
keywords:
description:
photos:
---
# Apache Arrow
Apache Arrow是一个内存分析技术的开发平台。
内存分析技术(In-Memorary Analytics)是从内存中直接获取分析数据，区别于传统工具在分析时是从存储在磁盘中的数据库，数据仓库或OLAP中获取数据。内存分析技术的数据，被预先载入内存，用户在执行查询及后续分析时，均直接从内存中获取数据。
Apache Arrow可以使的大数据系统更加快速的处理和转移数据。此外Apache Arrow规定了一种标准化不依赖于特定编程语言(language-independent)的内存列式(columnar memory)数据格式,用来处理平面数据(flat data)和分层数据(hierarchical data)。
Apache Arrow围绕的几个主题为：
* 零拷贝共享内存(Zero-copy shared memory)以及基于RPC的数据移动。
* 内存分析(in-memory analytics)和数据库查询(query processing).
* 读写文件格式(CSV,Apache ORC,Apache Parquet)。
Apache Parquet：是一种面向分析的，通用的列式存储格式，兼容各种数据处理框架，比如Spark，Hive，Impala，同时支持Avro，Thrift, Protocal Buffer等数据模型。

## 1. 数据格式：Arrow Columnar Format
* 和编程语言无关的内存数据结构
* 使用Google的Flatbuffers来进行元数据序列化
* 顺序访问数据邻接
* 随机访问时间复杂度为O(1)
* SIMD(Singel Instruction Multiple Data:单指令多数据流，能够复制多个操作数，并把它们打包在大型寄存器的一组指令集)以及向量化友好(vectorization-friendly)
* 允许真正的零拷贝共享内存(zero-copy in shared memory)，重定位(Relocatable：把程序的逻辑地址空间变换成内存中的实际物理地址空间，是实现多到程序在内存中同时运行的基础)而不用担心指针变换(pointer swizzling)。
pointer swizzling:指针变化，conversion of references based on name or position to direct pointer reference.内存中的节点通过逻辑指针(实际就是内存地址)连接，将这些节点保存到磁盘时，逻辑指针就没有任何意义了，需要变化一种方式来表示这些节点间的连接关系，这个变化的过程叫做unswizzling，反过来讲这些节点从磁盘中load到内存中变化就是swizzling。因为Apache Arrow是是内存数据，所以不存在pointer swizzling。

### 1.1 物理内存布局(Physical Memory Layout)
在物理内存中，一个数组由几段元数据和数据组成：
* 一种逻辑数据类型(logical data type)
每一种逻辑数据类型都有一个已经完全定义好的物理布局，其中logical data type包括下列几种类型：
 * Primitive(fixed-size):拥有相同字节数的值
 * Variable-size Binary:拥有不同字节长度的值，可以采用32位编码或者64位编码
 * Fixed-size List:嵌套型布局，每一个值都是由相同数量的元素组成，每一个元素都是从相同的子数列的数据类型继承而来。
 * Variable-size List:嵌套型布局，每一个值都是由不同数量的元素组成，每一个元素都是从相同的子数列的数据类型继承而来。
 * Struct:嵌套型数据结构，每一个值由不同的子域(child field)组成，子域的数据类型可以不同，但是长度需要相同。
 * Sparse Union / Dense Union:嵌套型数据结构
 * Null:一系列空值
* 一系列缓冲区(a sequence of buffer)
* 64字节长的有符号整型数据(64bit signed integer)
* 64字节长的有符号整型数据用来表示null值数量。
* 可选：一个字典，对于字典编码(dictionry-encoded)的数组
对于嵌套型的数组(nested arrays),就是有一系列的一个或多个上述数据结构，称为子数组(child arrays)。
Apache Arrow内存列式数据结构只应用在数据上，而并非元数据。内存元数据的数据格式可根据具体实现方式而改变，元数据的序列化使用Flatbuffer，和实现方式无关(implementation-independent)。

#### 1.1.1 缓存区对齐和填充(Buffer Alignment and Padding)
建议在实现时，分配内存的时候保证地址对齐(8-或64-字节的倍数)，并且在长度上进行填充到(8-或64-字节的倍数)。当在IPC(interprocess communication：进程间通信)序列化Arrow data时，对于地址对齐和长度填充是强制的。建议使用64字节对齐和填充。
#### 1.1.2 数组长度
数组长度作为数据的元数据(Arrow Metadata)，用64字节长的有符号整型数据表示。在跨语言环境中，建议将数组长度保持在2^31 - 1，大于此的数据集可以用过个数组来表示。
#### 1.1.3 Null值数量
null值数量也属于元数据，用64字节的有符号整型数据表示，可能和数组的程度一样大。
#### 1.1.4 有效性位图(validity bitmap)
指示每一位的值是否有效(及是否为null值)，采用的是LSB，每8个字节从右往左读。
一个没有null值的数组，在实现的时候可以选择不分配内尊空间给有效性位图，当然为了实现的方便，也可以无论是否需要都为有效性位图分配空间。无论哪种方式，在共享内存的时候应当注意是否分配了该空间以及如何分配。
嵌套式数组有自己独立的null count以及validity bitmap，与它子数组的这两个值是独立分开的。
#### 1.1.5 举例1
以一个int32 Array为例，一个数字占4个字节
![Screen Shot 2020-03-02 at 4.42.57 P](media/Screen%20Shot%202020-03-02%20at%204.42.57%20PM.png)
总结：primitive array型的数据结构由4部分组成，数组长度，null值数量，缓存区，其中缓存区由value buffer以及validity bitmap buffer组成。
#### 1.1.6 举例2
如果是不同长度的的列表嵌套式数组(Variable-size List Array)或者不同长度的二进制数组(Variable-size Binary Layout),及每一个数组的值可能由0或多个byte组成，这种情况下，Value Buffer将会拆分成两个部分，offsets buffer以及data buffer。
offset buffer拥有length+1 signed integer(int32或int64，根据数据类型而定)，编码了data buffer里每一个slot的开始位置。
the position and length of slot j is computed as:
```slot_position = offsets[j]```
```slot_length = offsets[j + 1] - offsets[j]  // (for 0 <= j < length)```
下面以一个List\<int8> array为例：
![Screen Shot 2020-03-02 at 5.27.25 P](media/Screen%20Shot%202020-03-02%20at%205.27.25%20PM.png)
buffer的组成有validity map, offsetmap,而它的值，其实是之前一个primitive array组成，primitive array的在buffer组成是validity map和value buffer，因为没有空值，validity map省略，只有value map。
![Screen Shot 2020-03-02 at 6.05.09 P](media/Screen%20Shot%202020-03-02%20at%206.05.09%20PM.png)
### 1.2 序列化和进程间通信Serialization and Interprocess Communication(IPC)
序列化的内存列式数据的主要单位是"record batch"。一个record batch就是一组有序的数组(ordered collection of arrays)，这些有序的数组被称作这个”record batch“的”fields“。每一个数组的长度都是相同的，但是数据类型可以不相同。一个record batch的每一个域的名字和数据类型，组成了这个record batch的schema。
Apache Arrow定义了一个协议用来把这些record batches编码成二进制的payload用于传输，并且可以将这些payload重新构建成record batch而无需复制内存(memory copy)。
Apache arrow的这个协议把schema，
RecordBatch或者DictionaryBatch利用一个单项流式二进制信息(a one-way stream of binary messages)表示出来，这种信息，叫做压缩式IPC信息(encapsulated IPC message)。
#### 1.2.1 压缩式IPC信息格式
压缩式IPC信息可以用于简单流式传输(simple streaming)和基于文件的序列化(file-based serialization),压缩式IPC信息可以通过它的元数据信息就可以”反序列化“为内存Arrow数组，而无需复制或移动实际的数据。
压缩式IPC格式如下：
* 一个32位的标识符。```0xFFFFFFFF```表示该信息为有效信息。这个部分是为了适配Flatbuffer。
* 一个32位的小端长度前缀表示元数据的大小。(\<metadata_size>)
* 信息的元数据。(\<metadata_flatbuffer>)
* 填充字节，为了满足8字节字节边界(\<padding>)
* 消息体，可选(\<meassage body> optional)

最后的格式如下：
![Screen Shot 2020-03-02 at 7.32.07 P](media/Screen%20Shot%202020-03-02%20at%207.32.07%20PM.png)
最后整个消息的大小一定要为8字节的倍数。
其中```metadata_size```包括信息以及padding的大小，```metadata_flatbuffer```包括序列化的信息Flatbuffer值。Flatbuffer值包含下列信息：在message.fbs中定义
* 版本号
* 特定信息值(schema，RecordBatch，还是DictionaryBatch)。
* 信息本体的大小
* 自定义元数据信息(custom_metadata)。


压缩IPC信息又分为Schema message, RecordBatch message,  DictionaryBatch message.
* Schema message：本身不含有任何数据，只有它所定义的RecordBatch的元数据。Schema中包含的是RecordBatch所包含的field的信息，在Flatbuffer序列化的时候field被序列化为Field FlatBuffer type，其中包含
  *  field名称
  *  field数据类型
  *  是否可为null
  *  如果是嵌套型，则会有所有子field的值
  *  是否为dictionary-encoded
  
* RecordBatch message：包含实际的由schema定义的在物理内存上的data buffer。RecordBatch的元数据提供了每个buffer的地址和大小，允许使用指针运算直接将data buffer构建回数组而不需内存拷贝。RecordMessage的信息体构成如下：
  * 数据头(data header)，定义为RecordBatch型数据。还包含每一个展开的field的长度以及null的数量。以及每一块内存的偏移量(offset)和长度。
  * 数据体(body),一系列展开的(非嵌套)memory buffers以及padding来确保是8字节的倍数。
  
![Screen Shot 2020-03-03 at 3.51.04 P](media/Screen%20Shot%202020-03-03%20at%203.51.04%20PM.png)
![Screen Shot 2020-03-03 at 3.51.11 P](media/Screen%20Shot%202020-03-03%20at%203.51.11%20PM.png)
![Screen Shot 2020-03-03 at 3.51.18 P](media/Screen%20Shot%202020-03-03%20at%203.51.18%20PM.png)

#### 1.2.2 IPC Streaming Format
Apache Arrow对IPC有一个流式传输协议。流式传输主要就是一系列遵循上述格式的IPC Message的集合。一般来说，流的开端是Schema类型的IPC Message，这个Schema之后的所有的RecordBatch类型的IPC Message全部都符合该Schema。如果存在有Dictionary-encoded情况，Dictionary类型的IPC Message会和RecordBatch Message都会在数据流中，但是任何用到Dictionary-Key的RecordBatch Message之前，存有该key信息的Dictionary Message一定已经出现了。
#### 1.2.3 IPC File Format
IPC File Format主要是用来支持随机访问(Random Access)。文件其实也是由流式格式组成，只是在文件的开头和结尾都有一个字符串```ARROW1(plus padding)```。两个字符串之间的内容和流式传输的格式完全一样。在文件结尾，有一个\<footer>，由Schema Message的副本和内存的偏移及长度组成，内存的偏移及长度用来描述文件中的每一个data block(RecordBatch)。
![Screen Shot 2020-03-03 at 4.28.38 P](media/Screen%20Shot%202020-03-03%20at%204.28.38%20PM.png)

## 2. Arrow Flight RPC
Arrow Flight RPC是一个基于Arrow Data高性能的数据服务RPC框架，它也是在gRPC和IPC格式的基础上构建的。
Methods和rpc message由Protobuf定义，客户端即使不直接支持Arrow Flight，只要可以分别支持gRPC和Apache Arrow即可，不过Flight对Protobuf在内存性能上进行了更深层的优化。
### 2.1 RPC Method
Flight定义了一系列RPC方法用来上传/下载数据，可以获取一个数据流对应的元数据，列出当前可行的全部数据流，还可以根据具体的应用定义对应的RPC方法。一个Flight的客户端可以连接任意一个Flight的服务端并执行基本操作。
数据流由标识符进行标识，标识符可以是路径或者任意一个二进制命令。
如果一个客户端希望请求下载数据，需要下面几步：
* 构造或获取一个目标数据集的标识符，可以通过listFlights来查看当前available的数据流的标识符。
* Call```GetFlightInfo(FlightDescriptor)```来获取```FlightInfo```信息，这个信息里包含详细的信息，关于数据集的位置，schema以及估算的大小等元数据信息。Flight不要求这些元数据和他们所描述的数据本身在同一台服务器上：这个方法可以列出其他与之相连的服务端的服务器。```FlightInfo```信息包含一个```Ticket```，是一个不透明的二进制token，服务端需要这个token来识别客户端所请求的数据集。
* 如果需要，会与其他Server连接
* Call```DoGet(Ticket)```来获取recordbacth的数据流。

如果客户需要上传数据：
* 构建或获取一个标识符。
* Call```DoPut(FlightData)```来上传一个由RecordBacth构成的数据流，在第一跳信息中心需要包含标识符(FlightDescriptor)。

## Question
Implementation guidelines
An execution engine (or framework, or UDF executor, or storage engine, etc) can implement only a subset of the Arrow spec and/or extend it given the following constraints:

Implementing a subset the spec
If only producing (and not consuming) arrow vectors: Any subset of the vector spec and the corresponding metadata can be implemented.

If consuming and producing vectors: There is a minimal subset of vectors to be supported. Production of a subset of vectors and their corresponding metadata is always fine. Consumption of vectors should at least convert the unsupported input vectors to the supported subset (for example Timestamp.millis to timestamp.micros or int32 to int64).
1. in java ```Class AllocationManager```:<br>Manages the relationship between one or more allocators and a particular **UDLE**（UnsafeDirectLittleEndian）. Ensures that one allocator owns the memory that multiple allocators may be referencing. Manages a BufferLedger between each of its associated allocators.<br>The only reason that this isn't package private is we're forced to put ArrowBuf in Netty's package which need access to these objects or methods.<br>Threading: AllocationManager manages thread-safety internally. Operations within the context of a single BufferLedger are lockless in nature and can be leveraged by multiple threads. Operations that cross the context of two ledgers will acquire a lock on the AllocationManager instance. Important note, there is one AllocationManager per UnsafeDirectLittleEndian buffer allocation. As such, there will be thousands of these in a typical query. The contention of acquiring a lock on AllocationManager should be very low.
