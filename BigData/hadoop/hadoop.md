## MapReduce的整体流程

1. 分片、格式化

   分片：将输入源文件在逻辑上划分为大小相等的数据分片（split），Hadoop会为每一个分片启动一个MapTask，并由该任务执行自定义的map()函数

   格式化：将划分好的分片内容转换为可以作为map输入的<key,value>键值对。其中key代表偏移量，value代表每一行内容。

2. 执行MapTask

   Read阶段：MapTask通过用户编写的RecordReader，从输入InputSplit解析出一个个key/value

   Map阶段：将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value

   Collect阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()收集结果。在该函数内部，它会调用Partitioner函数将生成的key/value分区，并写入一个环形内存缓冲区。

   Spill阶段：溢写阶段，当环形缓冲区满后，MapReduce会先将数据写到本地磁盘上，生成一个临时文件。

   Combine阶段：对所有临时文件进行一次合并以确保最终只会生成一个文件

3. 执行Shuffle过程

   Shuffle将MapTask输出的处理结果数据分发给ReduceTask，并在分发的过程中，对数据按key进行分区和排序。

4. 执行ReduceTask

   Copy阶段：Reduce会从各个MapTask上远程复制一片数据（每个MapTask传来的数据都是有序的），并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

   Merge阶段：在远程复制数据的同时，ReduceTask会启动两个后台线程，分别对内存和磁盘上的文件进行合并，以防止内存使用过多或者磁盘文件过多。

   Sort阶段：用户编写 reduce() 方法输入数据是按 key 进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现了对自己的处理结果局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

   Reduce阶段：对排序后的键值对调用reduce()方法，键相等的键值对调用一次reduce()方法，每次调用会产生零个或多个键值对，最后把这些输出的键值对写入到HDFS中

   Write阶段：write() 函数将计算结果写到 HDFS 上。

5. 写入文件

   将ReduceTask生成的<key,value>传入OutputFormat的write方法，实现文件的写入操作

## MapReduce的Shuffle过程

1. MapTask收集map()方法输出的kv对，调用Partitioner进行分区，放到内存缓冲区
2. 从内存缓冲区溢出本地磁盘文件
3. 多个溢出文件被合并成大的溢出文件
4. 在合并的过程中，针对key进行快速排序
5. ReduceTask抓取同一个分区的不同MapTask的结果文件，将这些结果文件进行归并排序
6. 合并成大文件后，Shuffle的过程就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）
