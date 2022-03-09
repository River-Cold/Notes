# Spark面试题

## Spark的定义

> Spark是一种基于内存的快速、通用、可扩展的大数据分析计算引擎

## Spark为什么比MapReduce快？

> 1. 基于内存计算，减少低效的磁盘交互；
>    1. mapreduce框架中，一个程序只能拥有一个map一个reduce的过程，如果运算逻辑很复杂，一个map+一个reduce是表述不出来的，可能就需要多个map-reduce的过程；MapReduce要实现多个map多个reduce通常需要将计算的中间结果写入磁盘，然后还要读取磁盘，导致了频繁的磁盘IO。
>    2. Spark则不需要将计算的中间结果写入磁盘，这得益于Spark的RDD（弹性分布式数据集，很强大）和DAG（有向无环图），其中DAG记录了job的stage以及在job执行过程中父RDD和子RDD之间的依赖关系。中间结果能够以RDD的形式存放在内存中，且能够从DAG中恢复，大大减少了磁盘IO。
> 2. 高效的调度算法，基于DAG；
> 3. 容错机制Linage

## Spark的基于内存计算是什么含义？

> Spark的基于内存计算是指多个作业之间的数据通信不需要借助硬盘而是通过内存。

## 参考文献

[(15 封私信 / 80 条消息) 为什么Spark比MapReduce快？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/31930662)