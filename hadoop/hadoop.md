# Hadoop

**Hadoop的核心架构**

Hadoop的核心，说白了，就是HDFS和MapReduce。HDFS为海量数据提供了**存储**，而MapReduce为海量数据提供了**计算框架**。



![img](https://pic2.zhimg.com/80/v2-af31c33db7daa0761da1ed03327154fd_720w.jpg)Hadoop核心架构



让我们来仔细看看，它们分别是怎么工作的。

#### **HDFS**：

整个HDFS有三个重要角色：**NameNode**（名称节点）、**DataNode**（数据节点）和**Client**（客户机）。



![img](https://pic4.zhimg.com/80/v2-12bac7206f243ab217e58a23a555da47_720w.jpg)典型的主从架构，用TCP/IP通信



**NameNode：**是Master节点（主节点），可以看作是分布式文件系统中的管理者，主要负责管理文件系统的命名空间、集群配置信息和存储块的复制等。NameNode会将文件系统的Meta-data存储在内存中，这些信息主要包括了文件信息、每一个文件对应的文件块的信息和每一个文件块在DataNode的信息等。



**DataNode：**是Slave节点（从节点），是文件存储的基本单元，它将Block存储在本地文件系统中，保存了Block的Meta-data，同时周期性地将所有存在的Block信息发送给NameNode。



**Client：**切分文件；访问HDFS；与NameNode交互，获得文件位置信息；与DataNode交互，读取和写入数据。 



还有一个**Block（块）**的概念：Block是HDFS中的基本读写单元；HDFS中的文件都是被切割为block（块）进行存储的；这些块被复制到多个DataNode中；块的大小（通常为64MB）和复制的块数量在创建文件时由Client决定。



我们来简单看看HDFS的读写流程。



首先是**写入流程**：



![img](https://pic1.zhimg.com/80/v2-cbf6dfb751bcf61d74726948f3df550c_720w.jpg)



1 用户向Client（客户机）提出请求。例如，需要写入200MB的数据。

2 Client制定计划：将数据按照64MB为块，进行切割；所有的块都保存三份。

3 Client将大文件切分成块（block）。

4 针对第一个块，Client告诉NameNode（主控节点），请帮助我，将64MB的块复制三份。

5 NameNode告诉Client三个DataNode（数据节点）的地址，并且将它们根据到Client的距离，进行了排序。

6 Client把数据和清单发给第一个DataNode。

7 第一个DataNode将数据复制给第二个DataNode。

8 第二个DataNode将数据复制给第三个DataNode。

9 如果某一个块的所有数据都已写入，就会向NameNode反馈已完成。

10 对第二个Block，也进行相同的操作。

11 所有Block都完成后，关闭文件。NameNode会将数据持久化到磁盘上。



**读取流程：**





![img](https://pic2.zhimg.com/80/v2-70f0c5acbc21cfacae10c19981522395_720w.jpg)





1 用户向Client提出读取请求。

2 Client向NameNode请求这个文件的所有信息。

3 NameNode将给Client这个文件的块列表，以及存储各个块的数据节点清单（按照和客户端的距离排序）。

4 Client从距离最近的数据节点下载所需的块。



（注意：以上只是简化的描述，实际过程会更加复杂。）



### MapReduce：



MapReduce其实是一种编程模型。这个模型的核心步骤主要分两部分：**Map（映射）**和**Reduce（归约）**。



当你向MapReduce框架提交一个计算作业时，它会首先把计算作业拆分成若干个**Map任务**，然后分配到不同的节点上去执行，每一个Map任务处理输入数据中的一部分，当Map任务完成后，它会生成一些中间文件，这些中间文件将会作为**Reduce任务**的输入数据。Reduce任务的主要目标就是把前面若干个Map的输出汇总到一起并输出。



![img](https://pic3.zhimg.com/80/v2-eb95cb2b3b945f38e89758e9f8ecebb6_720w.jpg)



是不是有点晕？我们来举个例子。



![img](https://pic1.zhimg.com/80/v2-42b95bf6958ee05771bddfdf0c48ac60_720w.jpg)



上图是一个统计词频的任务。



1 Hadoop将输入数据切成若干个分片，并将每个split（分割）交给一个map task（Map任务）处理。

2 Mapping之后，相当于得出这个task里面，每个词以及它出现的次数。

3 shuffle（拖移）将相同的词放在一起，并对它们进行排序，分成若干个分片。

4 根据这些分片，进行reduce（归约）。

5 统计出reduce task的结果，输出到文件。



如果还是没明白的吧，再举一个例子。



一个老师有100份试卷要阅卷。他找来5个帮手，扔给每个帮手20份试卷。帮手各自阅卷。最后，帮手们将成绩汇总给老师。很简单了吧？



MapReduce这个框架模型，极大地方便了编程人员在不会分布式并行编程的情况下，将自己的程序运行在分布式系统上。



哦，差点忘了，在MapReduce里，为了完成上面这些过程，需要两个角色：**JobTracker**和**TaskTracker**。



![img](https://pic1.zhimg.com/80/v2-624d63d33f832cbb64235f23ad22809c_720w.jpg)



JobTracker用于调度和管理其它的TaskTracker。JobTracker可以运行于集群中任一台计算机上。TaskTracker 负责执行任务，必须运行于 DataNode 上。



HDFS和MR共同组成Hadoop分布式系统体系结构的核心。HDFS在集群上实现了分布式文件系统，MR在集群上实现了分布式计算和任务处理。HDFS在MR任务处理过程中提供了文件操作和存储等支持，MR在HDFS的基础上实现了任务的分发、跟踪、执行等工作，并收集结果，二者相互作用，完成分布式集群的主要任务。



### **Hbase**：

Hive是建立在Hadoop上的数据仓库基础架构。它提供了一系列的工具，用来进行数据提取、转换、加载，这是一种可以存储、查询和分析存储在Hadoop中的大规模数据机制。可以把Hadoop下结构化数据文件映射为一张成Hive中的表，并提供类sql查询功能，除了不支持更新、索引和事务，sql其它功能都支持。可以将sql语句转换为MapReduce任务进行运行，作为sql到MapReduce的映射器。提供shell、JDBC/ODBC、Thrift、Web等接口。优点：成本低可以通过类sql语句快速实现简单的MapReduce统计。作为一个数据仓库，Hive的数据管理按照使用层次可以从元数据存储、数据存储和数据交换三个方面介绍。

（1）元数据存储

Hive将元数据存储在RDBMS中，有三种方式可以连接到数据库：

·内嵌模式：元数据保持在内嵌数据库的Derby，一般用于单元测试，只允许一个会话连接

·多用户模式：在本地安装Mysql，把元数据放到Mysql内

·远程模式：元数据放置在远程的Mysql数据库

（2）数据存储

首先，Hive没有专门的数据存储格式，也没有为数据建立索引，用于可以非常自由的组织Hive中的表，只需要在创建表的时候告诉Hive数据中的列分隔符和行分隔符，这就可以解析数据了。

其次，Hive中所有的数据都存储在HDFS中，Hive中包含4中数据模型：Tabel、ExternalTable、Partition、Bucket。

Table：类似与传统数据库中的Table，每一个Table在Hive中都有一个相应的目录来存储数据。例如：一个表zz，它在HDFS中的路径为：/wh/zz，其中wh是在hive-site.xml中由$指定的数据仓库的目录，所有的Table数据（不含External Table）都保存在这个目录中。

Partition：类似于传统数据库中划分列的索引。在Hive中，表中的一个Partition对应于表下的一个目录，所有的Partition数据都存储在对应的目录中。例如：zz表中包含ds和city两个Partition，则对应于ds=20140214，city=beijing的HDFS子目录为：/wh/zz/ds=20140214/city=Beijing;

Buckets：对指定列计算的hash，根据hash值切分数据，目的是为了便于并行，每一个Buckets对应一个文件。将user列分数至32个Bucket上，首先对user列的值计算hash，比如，对应hash=0的HDFS目录为：/wh/zz/ds=20140214/city=Beijing/part-00000;对应hash=20的，目录为：/wh/zz/ds=20140214/city=Beijing/part-00020。

ExternalTable指向已存在HDFS中的数据，可创建Partition。和Table在元数据组织结构相同，在实际存储上有较大差异。Table创建和数据加载过程，可以用统一语句实现，实际数据被转移到数据仓库目录中，之后对数据的访问将会直接在数据仓库的目录中完成。删除表时，表中的数据和元数据都会删除。ExternalTable只有一个过程，因为加载数据和创建表是同时完成。世界数据是存储在Location后面指定的HDFS路径中的，并不会移动到数据仓库中。

（3）数据交换

·用户接口：包括客户端、Web界面和数据库接口

·元数据存储：通常是存储在关系数据库中的，如Mysql，Derby等

·Hadoop：用HDFS进行存储，利用MapReduce进行计算。

关键点：Hive将元数据存储在数据库中，如Mysql、Derby中。Hive中的元数据包括表的名字、表的列和分区及其属性、表的属性（是否为外部表）、表数据所在的目录等。

Hive的数据存储在HDFS中，大部分的查询由MapReduce完成。

