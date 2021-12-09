# 大数据面试题（By RiverCold）

## Hadoop

### Hdaoop运行模式

> [Hadoop运行模式](https://blog.csdn.net/zane3/article/details/79829175)包括：本地模式、伪分布式模式以及完全分布式模式。
>
> 1. 本地模式：单机运行。使用本地文件系统和本地MapReduce运行器，不需要HDFS和YARN守护进程。
> 2. 伪分布式模式：单机运行。Hadoop守护进程运行在本地机器上，模拟一台机器的Hadoop集群伪分布式是完全分布式的一个特例。
> 3. 完全分布式模式：多台服务器组成分布式环境Hadoop守护进程运行在一个集群上。

### Hadoop1.x、2.x、3.x区别

> 1. Hadoop1.x时代：Hadoop中的MapReduce同时处理业务计算和资源调度，耦合性较大。
> 2. Hadoop2.x时代：增加了YarnMapReduce只负责业务计算，Yarn只负责资源调度。
> 3. Hadoop3.x时代：相较于Hadoop2.x时代在组成上没有变化，但较大优化了已有组件，引入了新的功能。

### Hadoop三大基本组件

> 1. [HDFS](https://baike.baidu.com/item/hdfs)：Hadoop Distributed File System，简称HDFS，即Hadoop的分布式文件系统，负责海量数据的存储。
> 2. [YARN](https://baike.baidu.com/item/yarn)：Yet Another Resource Negotiator，简称YARN，即Hadoop的资源管理器，负责海量数据计算时的资源调度。
> 3. [MapReduce](https://baike.baidu.com/item/MapReduce/133425?fr=aladdin)：MapReduce将计算过程分为两个阶段Map和Reduce，即Hadoop的并行计算系统，负责海量数据的并行计算。
>    1. Map（映射）阶段并行处理输入数据
>    2. Reduce（归约）阶段对Map结果进行汇总
>

### Hadoop生态圈

> Hadoop生态圈包括Hadoop框架本身和保证hadoop框架正常高效运行的其他框架。根据服务对象和层次可以分为：数据来源层、数据传输层、数据存储层、资源管理层、数据计算层、任务调度层、业务模型层
>
> ![](https://cdn.nlark.com/yuque/0/2021/png/21953026/1629386282142-70da2bfc-7170-4d52-8061-59ebaf8446d3.png)
>
> 图中涉及的技术名词解释如下：
>
> 1. Sqoop（数据传递工具）：是SQL-to-Hadoop的缩写，主要用于在Hadoop(Hive)与传统的数据库(mysql、postgresql...)间进行数据的传递，可以将一个关系型数据库（例如 ： MySQL ,Oracle 等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中
> 2. Flume（日志收集工具）：一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统
> 3. HBase（分布式列存储数据库）：一个分布式、面向列的开源数据库HBase不同于一般的关系型数据库，它是一个适合于非结构化数据存储的数据库
> 4. Hive（数据仓库工具）：基于Hadoop的一个数据仓库工具，可以把结构化的数据文件映射成一张数据库表，并提供简单的SQL查询功能，可以将SQL语句转换为MapReduce任务进行运行适合数据仓库的统计分析
> 5. Spark（分布式计算框架）：当前最流行的开源大数据内存计算框架可以基于Hadoop上存储的大数据进行计算不同于MapReduce的是Job中间输出结果可以保存在内存中，从而不再需要读写HDFS，因此Spark能更好地适用于数据挖掘与机器学习等需要迭代计算的算法
> 6. Flink（分布式计算框架）：当前最流行的开源大数据内存计算框架用于实时计算的场景较多
> 7. Oozie（工作流调度器）：一个管理Hadoop作业（job）的工作流程调度管理系统
> 8. Zookeeper（分布式协作服务）：一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、分布式服务、组服务等
>

## HDFS

### HDFS默认数据块的大小是多少？为什么？

> HDFS中的文件在物理上是分块存储（block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认块大小在Hadoop2.x版本中是128M，老版本中是64M
>
> HDFS的块大小取决于磁盘传输速率目前磁盘传输率约为100M/s，而HDFS读取文件时最佳的寻址时间为10ms，理论上寻址时间为传输时间的1%时最佳
>
> 故最佳传输时间为
> $$
> 1ms×1000=1000ms=1s
> $$
> 块的最佳大小=最佳传输时间×磁盘传输速率：
> $$
> 1s×100M/s = 100M
> $$
> 所以定义块大小为128M
>

### 为什么HDFS块的大小不能设置太小，也不能设置太大？

> 如果HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置；
>
> 如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间导致程序在处理这块数据时，会非常慢；
>
> 总结：HDFS块的大小设置主要取决于磁盘的传输速率
>

### HDFS组成架构？

> ![img](https://cdn.nlark.com/yuque/0/2021/png/21953026/1631347406286-eed355fb-e2b1-4249-8e84-528c3f7b24f2.png)
>
> 架构由4个部分组成，分别为HDFS Client、NameNode、DataNode和Secondary NameNode
>
> 1. Client：客户端
>    1. 文件切分文件上传HDFS的时候，客户端将文件切分成一个一个的Block，然后进行上传
>    2. 与NameNode交互，获取文件的位置信息；
>    3. 与DataNode交互，读取或者写入数据；
>    4. 提供一些命令来管理HDFS，比如NameNode格式化；
>    5. 通过一些命令来访问HDFS，比如对HDFS增删改查操作；
> 2. NameNode：就是Master，它是一个主管、管理者
>    1. 管理HDFS的名称空间；
>    2. 配置副本策略；
>    3. 管理数据块（Block）映射信息；
>    4. 处理客户端读写请求；
> 3. DataNode：就是Slave，NameNode下达命令，DataNode执行实际操作
>    1. 存储实际的数据块；
>    2. 执行数据块的读/写操作；
> 4. Secondary NameNode：并非NameNode的热备份当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务
>    1. 辅助NameNode，分担其工作量；
>    2. 在紧急情况下，可辅助恢复NameNode；
>

### HDFS的写数据流程？

> ![img](https://cdn.nlark.com/yuque/0/2021/png/21953026/1631349353957-8a9426dd-7a91-41b0-8775-fde95c162a13.png)
>
> 1. 客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否存在，父目录是否存在
> 2. NameNode返回是否可以上传
> 3. 客户端请求第一个Block上传到哪几个datanode服务器上
> 4. NameNode返回3个DataNode节点，分别为dn1、dn2、dn3
> 5. 客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。
> 6. dn1、dn2、dn3逐级应答客户端
> 7. 客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存，以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答）
> 8. 当第一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务（重复执行3-7步）

### HDFS的读数据流程？

> ![img](https://cdn.nlark.com/yuque/0/2021/png/21953026/1631460280331-5e9c5202-71a1-4a1a-a1b8-6c5a188de260.png?x-oss-process=image%2Fresize%2Cw_902%2Climit_0)
>
> 1. 客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址
> 2. 挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据
> 3. DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）
> 4. 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件

### NN和2NN工作机制（了解）

> ![img](https://cdn.nlark.com/yuque/0/2021/png/21953026/1631626177178-b9e3267a-e398-4d91-98fe-ee44df7183cb.png)
>
> **第一阶段：NameNode启动**
>
> 1. 第一次启动NameNode格式化，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
> 2. 客户端对元数据进行增删改的请求。
> 3. NameNode记录操作日志，更新滚动日志。
> 4. NameNode在内存中对元数据进行增删改。
>
> **第二阶段：Secondary NameNode工作**
>
> 1. Secnodary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果。
> 2. Secondary NameNode请求执行CheckPoint。
> 3. NameNode滚动正在写的Edits日志。
> 4. 将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。
> 5. Secondary NameNode加载编辑日志和镜像文件到内存，并合并。
> 6. 生成新的镜像文件fsimage.chkpoint。
> 7. 拷贝fsimage.chkpoint到NameNode。
> 8. NameNode将fsimage.chkpoint重新命名成fsimage。

### DataNode工作机制（了解）

> ![img](https://cdn.nlark.com/yuque/0/2021/png/21953026/1631540997938-34f4d0f3-af51-4a61-94f4-be7993ff6ee5.png?x-oss-process=image%2Fresize%2Cw_1182%2Climit_0)
>
> 1. 一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的检验和，时间戳。
>
> 2. DataNode启动后向NameNode注册，通过后，周期性（6小时）的向NameNode上报所有的块信息。
>
>    DN向NN汇报当前解读信息的时间间隔，默认6小时
>
>    DN扫描自己节点块信息列表的时间，默认6小时
>
>    1. 心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。
>    2. 集群运行中可以安全加入和退出一些机器。

## MapReduce

### 谈谈Hadoop序列化和反序列化以及自定义bean对象实现序列化

> **序列化和反序列化**
>
> 1. 序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输
>
> 2. 反序列化就是将受到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。
>
> **为什么要序列化？**
>
> 一般来说，“活的”对象只生存在内存里，关机断电就没有了。而且“活的”对象只能由本地的进程使用，不能被发送到网络的另外一台计算机。然后序列化可以存储“活的”对象，可以将“活的”对象发送到远程计算机。
>
> **为什么不用Java序列化？**
>
> Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，head，继承体系等），不便于在网络中高效传输。所以，hadoop自己开发了一套序列化机制（Writable），精简高效。
>
> **自定义bean对象序列化传输步骤及注意事项**
>
> 1. 必须实现Writable接口
> 2. 反序列化时，需要反射调用空参构造函数，所以必须有空参构造
> 3. 重写序列化方法
> 4. 重写反序列化方法
> 5. 注意反序列化的顺序和序列化的顺序完全一致
> 6. 要想把结果显示在文件中，需要重写toString()，且用"\t"分开，方便后续使用
> 7. 如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序

### FileInputFormat切片机制

> ```
> waitForCompletion()
> submit()
> 1、建立连接
> 	connect()
> 	1、创建提交job的代理
> 		new Cluster(getConfiguration())
> 	2、判断是本地yarn还是远程
> 		initialize(jobTrackAddr,conf)
> 2、提交job
> submitter.submitJobInternal(Job.this, cluster)	
> 	1、创建给集群提交数据的stag路径
> 	Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
> 	2、获取jobid，并创建job路径
> 	JobID jobid = submitClient.getNewJobID();
> 	3、拷贝jar包到集群
> 	copyAndConfigureFiles(job, submitJobDir);
> 	rUploader.uploadFiles(job, jobSubmitDir);
> 	4、计算切片,生成切片规划文件
> 	writeSplits(job, submitJobDir)
> 	maps = writeNewSplits(job, jobSubmitDir);
> 	input.getSplits(job);
> 	5、向stag路径写xml配置文件
> 	writeConf(conf, submitJobFile);
> 	conf.writeXml(out);
> 	6、提交job，返回提交状态
> 	status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
> ```
>

### 在一个运行的Hadoop任务中，什么是InputSplit?

> FileInputFormat源码解析（input.getSplits(job))
>
> 1. 找到数据存储的目录
> 2. 开始遍历处理（规划切片）目录下的每一个文件
> 3. 遍历第一个文件（假设为ss.txt）
>    1. 获取文件大小`fs.sizeOf(ss.txt)`
>    2. 计算切片大小`computeSliteSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M`
>    3. 默认情况下，切片大小=blocksize
>    4. 开始切，形成3个切片（每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片）
>       1. 第1个切片：ss.txt—0-128M
>       2. 第2个切片：ss.txt—128-256M
>       3. 第3个切片：ss.txt—256M-300M
>    5. 将切片信息写到一个切片规划文件中
>    6. 整个切片的核心过程在getSplit()方法中完成
>    7. 数据切片只是在逻辑上对输入数据进行分片，并不会在磁盘上将其切分成分片进行存储。InputSplit只记录了分片的元数据信息，比如起始位置、长度以及所在的节点列表等。
>    8. 注意：block是HDFS上物理上存储的数据，切片是对数据逻辑上的划分
> 4. 提交切片规划文件到yarn上，yarn上的MrAppMaster就可以根据切片规划文件计算开启maptask个数

### 如何判定一个job的map和reduce的数量？

> 1. map数量
>
>    `splitSize=max{minSize,min{maxSize,blockSize}}`
>
>    map数量由处理的数据分成的block数量决定$default\_num=total\_size/split\_size$
>
> 2. reduce数量
>
>    reduce的数量`job.setNumReduceTasks(x)`；x为reduce的数量。不设置的话默认为1

### MapTask的个数由什么决定？

> 一个job的map阶段MapTask并行度（个数），由客户端提交job时的切片个数决定

### MapTask和ReduceTask工作机制（MapReduce工作原理）

> **MapTask工作机制**
>
> 1. Read阶段：Map Task通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value
> 2. Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value
> 3. Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
> 4. Spill阶段：即"溢写"，当环形缓冲区满后，MapReduce会先将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。
> 5. Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

> **ReduceTask工作机制**
>
> 1. Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中
> 2. Sort阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因为，ReduceTask只需对所有数据进行一次归并排序即可。
> 3. Reduce阶段：reduce()函数将计算结果写到HDFS中

### 描述mapReduce有几种排序及排序发生的阶段？

> 1. 部分排序：MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部有序。
> 2. 全排序：最终输出结果只有一个文件，且文件内部有序。实现方式是只设置一个ReduceTask。但该方法在处理大型文件时效率极低，因为一台机器处理所有文件，完全丧失了MapReduce所提供的并行架构。
> 3. 辅助排序（GroupingComparator分组）：MapReduce框架在记录到达reducer之前按键对记录排序，但键所对应的值并没有被排序。甚至在不同的执行轮次中，这些值的排序也不固定，因为它们来自不同的map任务且这些map任务在不同轮次中完成时间各不相同。一般来说，大部分MapReduce程序会避免让reduce函数依赖于值的排序，但是，有时也需要通过特定的方法对键进行排序和分组等以实现对值的排序。
> 4. 二次排序：在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序
>
> **自定义排序WritableComparable**
>
> bean对象实现WritableComparable接口重写compareTo方法，就可以实现排序
>
> ```
> @Override
> public int compareTo(FlowBean o){
> 	// 倒序排序，从大到小
> 	return this.sumFlow > o.getSumFlow() ? -1 : 1;	
> }
> ```
>
> 排序发生的阶段
>
> 1. 一个是在map side发生在spill后partition前。
> 2. 一个是在reduce side发生在copy后reduce前。

### 描述mapReduce中shuffle阶段的工作流程，如何优化shuffle阶段

> 分区、排序、溢写，拷贝到对应reduce机器上，增加combiner，压缩溢写的文件

### 描述mapReduce中combiner的作用是什么，一般使用情景，哪些情况不需要combiner，以及combiner和reduce的区别？

> 1. Combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量。
>
> 2. Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟reducer的输入kv类型相对应。
>
> 3. Combiner和reducer的区别在于运行的位置。
>
>    Combiner是在每一个maptask所在的节点运行；
>
>    Reducer是接收全局所有Mapper的输出结果。

### Hadoop的缓存机制

> 分布式缓存一个最重要的应用就是在进行join操作的时候

### 如何使用mapReduce实现两个表的join?

> 1. reduce side join：在map阶段，map函数同时读取两个文件File1和File2，为了区分两种来源的key/value数据对，对每条数据打一个标签（tag），比如：tag0表示来自文件File1，tag2表示来自文件File2。
> 2. map side join：map side join是针对以下场景进行的优化：两个待连接表中，有一个表非常大，而另一个表非常小，以至于小表可以直接存放到内存中。这样，我们可以将小表复制多份，让每个map task内存中存在一份（比如存放到hash table中），然后只扫描大表：对于大表中的每一条记录key/value，在hash table中查找是否有相同的key的记录，如果有，则连接后输出即可。

### 什么样的计算不能用mr来提速？

> 1. 数据量很小
> 2. 繁杂的小文件
> 3. 索引是更好的存取机制的时候
> 4. 事务处理
> 5. 只有一台机器的时候

### ETL是哪三个单词的缩写？

> Extraction-Transformation-Loading的缩写，中文名称为数据提取，转换和加载

