### Spark and HDFS

My research on source code that helped to provide an overview of how Spark and Hadoop work together

Special thanks to [Jacek Laskowski](https://github.com/jaceklaskowski) and his online book [The Internals Of Apache Spark](https://github.com/japila-books/apache-spark-internals)


## Introduction

***BIG DATA ECOSYSTEM***

![Screenshot](https://github.com/cat-ai/spark-and-hdfs/blob/master/pic/Big-Data-Ecosystem.png)

**Distributed data storage layer:**

* [HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) - allows to distribute the storage of Big Data across cluster

**Distributed data processing layer:**

* [YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) - resource manager of cluster 
* [MESOS](http://mesos.apache.org/) - resource manager of cluster

**Execution engine layer:**

* [MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html#Overview) - is a programming model and an associated implementation for processing and generating large
                                                                                     data sets. Users specify a map function that processes a
                                                                                     key/value pair to generate a set of intermediate key/value
                                                                                     pairs, and a reduce function that merges all intermediate
                                                                                     values associated with the same intermediate key. 

* [Spark](https://spark.apache.org/) - MapReduce and its variants have been highly successful
                                       in implementing large-scale data-intensive applications
                                       on commodity clusters. However, most of these systems
                                       are built around an acyclic data flow model that is not
                                       suitable for other popular applications. 
                                       Spark supports scalability and fault tolerance of MapReduce. 
                                       To achieve these goals, Spark introduces abstraction called resilient distributed datasets ([RDDs](https://spark.apache.org/docs/latest/rdd-programming-guide.html))

* [TEZ](https://tez.apache.org/) - is an extensible framework for building high performance batch and interactive data processing applications, coordinated by [YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) in [Apache Hadoop](https://hadoop.apache.org/)


**Distributed data as RDB representation layer:**

* [Hive](https://hive.apache.org/) - data warehouse software facilitates reading, writing, and managing large datasets residing in distributed storage using SQL. Provides a simple language known as HiveQL similar to SQL for querying, data summarization and analysis. Hive makes querying faster through indexing
* [Pig](https://pig.apache.org/) - is a platform for analyzing large data sets that consists of a high-level language for expressing data analysis programs, coupled with infrastructure for evaluating these programs.

**Distributed data to transactional platform layer:**

* [HBase](https://hbase.apache.org/book.html#arch.overview) - is a type of "NoSQL" database. "NoSQL" is a general term meaning that the database isn’t an RDBMS which supports SQL as its primary access language, but there are many types of NoSQL databases: BerkeleyDB is an example of a local NoSQL database, whereas HBase is very much a distributed database. Technically speaking, HBase is really more a "Data Store" than "Data Base" because it lacks many of the features you find in an RDBMS, such as typed columns, secondary indexes, triggers, and advanced query languages, etc


**Job scheduler on cluster layer:**
* [Oozie](https://oozie.apache.org/) - is a workflow scheduler system to manage Apache Hadoop jobs. Workflows are expressed as Directed Acyclic Graphs. Oozie runs in a Java servlet container Tomcat and makes use of a database to store all the running workflow instances.
* [Airflow](https://airflow.apache.org/) - is a platform created by the community to programmatically author, schedule and monitor workflows.
* [NiFi](https://nifi.apache.org/) - supports powerful and scalable directed graphs of data routing, transformation, and system mediation logic
* [Zookeeper](https://zookeeper.apache.org/) - is an effort to develop and maintain an open-source server which enables highly reliable distributed coordination


**Data platform manager layer:**

* [Ambari](https://ambari.apache.org/) - is aimed at making Hadoop management simpler by developing software for provisioning, managing, and monitoring Hadoop clusters. Ambari provides an intuitive, easy-to-use Hadoop management web UI backed by its RESTful APIs
* [Hue](https://gethue.com/) - SQL Assistant for Databases & Data Warehouses
* [Cloudera Manager](https://www.cloudera.com/products/product-components/cloudera-manager.html) - The fastest way to get up and running with Hadoop and Cloudera Enterprise

## Hadoop Distributed File System (HDFS)

![Screenshot](https://github.com/cat-ai/spark-and-hdfs/blob/master/pic/HDFS-Arch.png)


[Spark](https://spark.apache.org/) was designed to read and write data from and to HDFS, as well as other storage systems, such as HBase and Amazon’s S3. As such, Hadoop users can enrich their processing capabilities by combining Spark with Hadoop MapReduce, HBase, and other big data frameworks.

HDFS is a block-structured file system where each file is divided into [blocks](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html#Data+Blocks) of a pre-determined size. 
HDFS is designed to support very large files. Applications that are compatible with HDFS are those that deal with large data sets.
HDFS follows a master-slave architecture, where a cluster comprises of a single [Name Node](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html#NameNode+and+DataNodes) (master node) and all the other nodes are [Data Nodes](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html#NameNode+and+DataNodes) (slave nodes), and [Secondary NameNode](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Secondary_NameNode) (checkpoint node) that performs regular checkpoints in HDFS. 

***HDFS architecture***

**[NameNode (Active)](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#NameNode_and_DataNodes)**

Is a very highly available server that manages the blocks present on the Data Nodes (slave nodes), File System Namespace and controls access to files by clients. 


* [FsImage](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Metadata_Disk_Failure) - one of central data structures of HDFS. Its stored as a file in the NameNode’s local file system too. Contains the complete state of the file system

* [EditLogs](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Metadata_Disk_Failure) - second central data structure of HDFS. Persistently record every change that occurs to file system metadata.

* [JobTracker](https://cwiki.apache.org/confluence/display/HADOOP2/JobTracker) - is resource management (managing the task trackers), tracking resource availability and task life cycle management (tracking its progress, fault tolerance etc.)


**[DataNode](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#NameNode_and_DataNodes)**

Data Node stores actual data and works as instructed by Name Node. Data Node is a block server that contains a small amount of metadata about the Data Node itself and its relationship to a cluster, by performing low-level read/write requests from the FS clients

* [Block](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Metadata_Disk_Failure) -  a file in HDFS contains one or more [blocks](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html#Data+Blocks). A block has one or multiple copies (called replicas, see [Data Replication](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html#Data+Replication)), based on the configured replication factor. A replica is stored on a volume of a Data Node, and different replicas of the same block are stored on different Data Nodes

* [HFile](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Metadata_Disk_Failure) - a file format for [HBase](https://hbase.apache.org/book.html#arch.overview). A file of sorted key/value pairs. Both keys and values are byte arrays.

* [TaskTracker](https://cwiki.apache.org/confluence/display/HADOOP2/TaskTracker) - constant communication with the [JobTracker](https://cwiki.apache.org/confluence/display/HADOOP2/JobTracker) signalling the progress of the task in execution



**[Secondary NameNode (Stand-By)](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#Secondary_NameNode)**

The NameNode stores modifications to the file system as a log appended to a native file system file, edits. 
Works concurrently with **[NameNode](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#NameNode_and_DataNodes)** as helper daemon. It is responsible for combining the EditLogs with FsImage from the Name Node.

* [FsImage](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Metadata_Disk_Failure) - it's the snapshot of the filesystem when Name Node started. Once it has new FsImage, it copies back to Name Node

* [EditLogs](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Metadata_Disk_Failure) - gets the EditLogs from the Name Node in regular intervals and applies to FsImage


**Replication management**

HDFS provides a reliable way to store huge data in a distributed environment as data blocks, and they are also replicated to provide fault tolerance. 
If we have a file of size 800 MB, HDFS default size of each block is 128 MB (since Hadoop 2.x.x; 64 MB in Hadoop 1.x.x), and replication factor is equal to 3, so we'll have 21 replicated blocks distributed on Data Nodes


## Hadoop data formats

Data formats widely used in Hadoop that can be split across multiple disks. Hadoop allows for storage of data in text, binary, etc.


* Row-oriented format

  ```Values for each row are stored contiguously in the file```  
  
  * [Avro]() - 
  

* Column-oriented format
  
  ```Rows in a file are broken up into row splits and each split is stored in column-oriented approach: row value stored in in column```    
 
 * [Parquet](https://parquet.apache.org/) - is a columnar storage format available to any project in the Hadoop ecosystem
   *![Screenshot](https://raw.github.com/apache/parquet-format/master/doc/images/FileLayout.gif)
     * Header
       * Magic numbers - 4 bytes "PAR1" (beginning)
     * Row Group consists of a column chunk for each column in the dataset
       * Column - a chunk of the data for a particular column
         * Page contains values for a particular column only
         * Column metadata
     * Footer
       * File metadata contains the locations of all the column metadata start locations
       * Footer length
       * Magic numbers - 4 bytes "PAR1" (end)
 
 * [ORC](https://orc.apache.org/) - columnar file format designed for Hadoop workloads. Improves performance when Hive is reading, writing, and processing data. [ACID](https://en.wikipedia.org/wiki/ACID) transactions are only possible when using ORC as the file format
   *![Screenshot](https://cwiki.apache.org/confluence/download/attachments/31818911/OrcFileLayout.png?version=1&modificationDate=1366430304000&api=v2)
     * Magic number "ORC"
     * Stripes - distributed unit
       * Index data includes min and max values for each column and the row positions within each column
       * Row data is used in table scans
       * Stripe footer contains a directory of stream locations
     * File footer
     * postscript
       

* For structured text data
  *[Avro]()


- when you need to process single N columns of single row at the same time, then use row-oriented file format; when you need to process small number of columns, then use column-oriented file format.
- column-oriented formats are not suited to streaming writes
- parquet compression and encoding algorithms are time and space consuming
- running a query on Parquet faster than on ORC
- ORC takes much less time on SELECT


## Apache Spark

![Screenshot](https://github.com/cat-ai/spark-and-hdfs/blob/master/pic/Spark-HDFS.png)


[Apache Spark](https://github.com/apache/spark) is a unified analytics engine for large-scale data processing, it follows master-slave architecture where we have one coordinator and multiple distributed workers. The coordinator is called Spark Driver and it communicates with all the workers.
The Spark Driver is the program that declares the [transformations](https://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations) and [actions](https://spark.apache.org/docs/latest/rdd-programming-guide.html#actions) on [RDDs](https://spark.apache.org/docs/latest/rdd-programming-guide.html#resilient-distributed-datasets-rdds) of data and submits such requests to the master. 

See the next: [spark cluster mode overview](https://spark.apache.org/docs/latest/cluster-overview.html)

***Spark components:***

**[Application](https://databricks.com/glossary/what-are-spark-applications)** - consist of a driver program and a set of executors

* [DAGScheduler](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala) - stage-oriented scheduler. First RDD will be created by reading data in parallel from HDFS to different partitions on different nodes. [Divides](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L532) the Job into several stages according to the wide or narrow depends of RDD. It transforms a logical execution plan (i.e. RDD lineage of dependencies built using RDD transformations) to a physical execution plan (using stages). Spark defines tasks that can be computed in parallel with the partitioned data on the cluster. 
  DAGScheduler [merges](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L449) some transformations together - multiple transformation can be combined into one stage. The end result of DAGScheduler is [TaskSet](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSet.scala)
  
  [TaskContext](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/TaskContext.scala) - contextual information about a task which can be [read](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/BarrierTaskContext.scala#L199) or mutated during execution
  
  [TaskSet](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSet.scala) represents the missing partitions (uncomputed (RDD blocks on [BlockManagers](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManager.scala) on executors)) of a stage that can be run right away based on the data that is already on the cluster, e.g. map output files from previous stages, though they may fail if this data becomes unavailable
  
  DAGScheduler utilizes [DAGSchedulerEventProcessLoop](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala) to process scheduler events. DAGSchedulerEventProcessLoop has an internal daemon thread, polling events from the queue, and once the thread takes and event, [onReceive](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L2243) will be called.
  DAGSchedulerEventProcessLoop [creates](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L1057) new stage, which input is the [last RDD](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L1047), then it creates [new job](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L1093) from this [stage](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L1053).
  [Visits](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L626) RDD lineage, and can [create shuffle map stage](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L388) or [prepends](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L646) RDD, it's depends on dependency(see [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L640) and  [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L645))
  
  [OutputCommitCoordinator](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/OutputCommitCoordinator.scala) is [called](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L1226) in DAGScheduler for all the stages.
  The coordinator lives on the driver and the executors can request its permission to commit. As tasks commit and are reported as completed successfully or unsuccessfully by the DAGScheduler, the commit coordinator is informed of the task completion events as well to update its internal state
  
  [MapOutputTracker](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/MapOutputTracker.scala) tracks (see [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/MapOutputTracker.scala#L482) and [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/MapOutputTracker.scala#L476)) the locations of the shuffle map outputs(see [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/MapOutputTracker.scala#L624) and [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/MapOutputTracker.scala#L694)). When a new ShuffleMapStage is [created](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L399), [containsShuffle](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/MapOutputTracker.scala#L536) method will be called
  
  * Job - is created when invoked an action on an RDD. Jobs are work submitted to Spark.
    * [ActiveJob](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ActiveJob.scala) - running job in the DAGScheduler. Jobs can be of two types: a result job, which computes a ResultStage to execute an action, or a map-stage job, which computes the map outputs for a ShuffleMapStage before any downstream stages are submitted
    
      * [Stage](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/Stage.scala) is a set of parallel tasks all computing the same function that need to run as part of a Spark job, where all the tasks have the same shuffle dependencies. Stage consists of tasks based on input data [partitions](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rdd/RDD.scala#L295)
        * [ResultStage](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ResultStage.scala) - final stage applies a function on some partitions of an RDD to compute the result of an action
        * [ShuffleMapStage](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ShuffleMapStage.scala) - intermediate stage that produces data for shuffle. When executed, they save map output files that can later be fetched by reduce tasks
        
      * [Task](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/Task.scala) is an operation on RDD(corresponds to a RDD partition), [executed](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/Executor.scala#L465) as a [single thread](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/Task.scala#L81) in an [Executor](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/Executor.scala)
        * [ResultTask](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ResultTask.scala) [sends](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ResultTask.scala#L75) back the output to the driver application 
        * [ShuffleMapTask](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ShuffleMapTask.scala) [divides](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/Task.scala#L81) the elements of an RDD into multiple buckets (based on a partitioner specified in the [ShuffleDependency](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/Dependency.scala#L71))
        * [BarrierTaskContext](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/BarrierTaskContext.scala) - a context with extra contextual info and tooling for tasks in a barrier stage
          
* [TaskScheduler](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskScheduler.scala) - low-level task scheduler interface, [schedules](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSchedulerImpl.scala#L220) tasks according to scheduling mode. Gets sets of tasks (as [TaskSets](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSet.scala)) usually representing [missing partitions](https://github.com/eBay/Spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ShuffleMapStage.scala#L89) of a particular stage, [submitted](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala#L1046) to it from the [DAGScheduler](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala) for each stage. TaskScheduler [launches](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSchedulerImpl.scala#L533) tasks through ClusterManager. TaskScheduler creates new [TaskSet](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSet.scala), submits it and [waits](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/SparkListenerBus.scala#L34) for SparkListenerStageCompleted event. When TaskScheduler [submits](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSchedulerImpl.scala#L230) a [TaskSet](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSet.scala), it also [creates](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSchedulerImpl.scala#L400) one [TaskSetManager](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSetManager.scala) to track that TaskSet.

  * [rootPool](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSchedulerImpl.scala#L173) - schedulable entities with scheduling mode(FIFO, FAIR)
       
  * [TaskSetManager](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSetManager.scala) - schedules the tasks within a single TaskSet in the TaskScheduler. The main interfaces to it are [resourceOffer](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSetManager.scala#L413), which asks the TaskSet whether it wants to run a task and [handleSuccessfulTask](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSetManager.scala#L710) / [handleFailedTask](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/TaskSetManager.scala#L791), which tells it that one of its tasks changed state.
 
  * [BarrierCoordinator](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/BarrierCoordinator.scala) - a coordinator that [handles](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/BarrierCoordinator.scala#L142) all global sync requests from BarrierTaskContext
   
* [BlockManager](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManager.scala) running on every node (driver and executors) which provides interfaces for [putting](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManager.scala#L319) and [retrieving](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManager.scala#L1180) blocks both locally and remotely into various stores ([memory](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/memory/MemoryStore.scala), [disk](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/DiskStore.scala), and [off-heap]()). 
  Allows to [get host-local shuffle block data](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/network/BlockDataManager.scala#L34), [get local block data](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/network/BlockDataManager.scala#L48) and [put the block locally](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/network/BlockDataManager.scala#L60).
  It provides a data storage (memory / file storage) interface. The block is essentially different from the HDFS block: in HDFS, large files are divided into blocks for storage, and the block size is fixed at 512M. And the block in Spark is the user's operating unit, one block corresponds to one block organized memory, a complete file or a range of files and there is no way to fix the size of each block
  
  * [BlockManagerMaster](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManagerMaster.scala) allows executors for sending block status updates to driver. Sends message to the corresponding [RpcEndpoint](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEndpoint.scala) by calling [askSync](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEndpointRef.scala#L87) method in [RpcEndpointRef](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEndpointRef.scala).
  It's just a connector to the (master) Driver program - communicator with the real Driver program. On Driver program and Executor, there is a BlockManagerMaster. [BlockManagerMasterHeartbeatEndpoint](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManagerMasterHeartbeatEndpoint.scala) [thread-safe]() separate heartbeat out of BlockManagerMasterEndpoint due to performance consideration. [BlockManagerMasterEndpoint](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManagerMasterHeartbeatEndpoint.scala) an endpoint on the [master](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkEnv.scala#L358) node to track statuses (see [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkEnv.scala#L358) and [this](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/BlockManagerMaster.scala#L100)) of all slaves block managers
  
    * [RpcEnv](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEnv.scala) will process messages sent from RpcEndpointRef or remote nodes, and deliver them to corresponding RpcEndpoint
      * [NettyRpcEnv](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/netty/NettyRpcEnv.scala) - [Netty](https://github.com/netty/netty) based RpcEnv starts the RPC server and register necessary endpoints
    * [RpcEndpointRef](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEndpointRef.scala) is a reference to a RpcEndpoint in a RpcEnv.
    * [RpcEndpoint](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEndpoint.scala) defines an RPC endpoint [receiving](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEndpoint.scala#L77) messages using callbacks (executing functions)  

* [CacheManager](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/CacheManager.scala) - in-memory cache (registry) for structured queries. Data is cached using byte buffers stored in an InMemoryRelation. Uses [CachedData](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/CacheManager.scala#L39) data structure for managing cached structured queries([list of cached plans as an immutable sequence](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/CacheManager.scala#L57))

* [HeartbeatReceiver](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/HeartbeatReceiver.scala) - lives in the Spark Driver program to receive [heartbeats](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/HeartbeatReceiver.scala#L42) from executors. It's a thread-safe endpoint processing of one message happens before processing of the next message by the same [ThreadSafeRpcEndpoint](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rpc/RpcEndpoint.scala#L148)

* [Driver Program]() - is a Java process. This is the process where the main() method of Scala, Java, Python program runs. It executes the user code and creates a SparkContext and SparkSession.
  
  * [SparkContext](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkContext.scala) - is a heart of Spark application. It is the main entry point to Spark functionality. SparkContext is a most important task for Spark Driver program and set up internal services and also constructs a connection to Spark execution environment
  
  * [SparkSession](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/SparkSession.scala) - is an entry point to Spark and creating a SparkSession instance would be the first statement you would write to program with RDD, DataFrame and Dataset
  
  * [ClusterManager](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/ExternalClusterManager.scala) - pluggable cluster management, dispatches work for the cluster, handles starting executor processes. ClusterManager allocates "containers" and asks manager (YARN, MESOS, KUBERNETES) to run the executors on selected "containers". Can be separate from the Spark driver program. This is the case when running Spark on Mesos or YARN
  
    * [MesosClusterManager](https://github.com/apache/spark/blob/master/resource-managers/mesos/src/main/scala/org/apache/spark/scheduler/cluster/mesos/MesosClusterManager.scala) - cluster manager for creation of Mesos [scheduler](https://github.com/apache/spark/blob/master/resource-managers/mesos/src/main/scala/org/apache/spark/scheduler/cluster/mesos/MesosClusterScheduler.scala) and Mesos backend(there are two types: [CoarseGrained](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/cluster/CoarseGrainedSchedulerBackend.scala) - [MesosCoarseGrainedSchedulerBackend](https://github.com/apache/spark/blob/master/resource-managers/mesos/src/main/scala/org/apache/spark/scheduler/cluster/mesos/MesosCoarseGrainedSchedulerBackend.scala) and deprecated (since Spark 2.0.0) [Fine-Grained](https://spark.apache.org/docs/latest/running-on-mesos.html#fine-grained-deprecated) - [MesosFineGrainedSchedulerBackend](https://github.com/apache/spark/blob/master/resource-managers/mesos/src/main/scala/org/apache/spark/scheduler/cluster/mesos/MesosFineGrainedSchedulerBackend.scala)). 
    
    * [KubernetesClusterManager](https://github.com/apache/spark/blob/master/resource-managers/kubernetes/core/src/main/scala/org/apache/spark/scheduler/cluster/k8s/KubernetesClusterManager.scala) - cluster manager for creating of Kubernetes [scheduler]() and [backend](https://github.com/apache/spark/blob/master/resource-managers/kubernetes/core/src/main/scala/org/apache/spark/scheduler/cluster/k8s/KubernetesClusterSchedulerBackend.scala)
    
    * [YarnClusterManager](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/scheduler/cluster/YarnClusterManager.scala) - cluster manager for creation of a YARN [scheduler](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/scheduler/cluster/YarnClusterScheduler.scala) and YARN [backend](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/scheduler/cluster/YarnClusterSchedulerBackend.scala). Can handle the YARN master URL only

**Worker** hold many executors, for many Spark Application. One Spark Application has executors on many workers

  * [Executor](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/Executor.scala) launched in coordination with the Cluster Manager. Reserve CPU and memory resources, allocates resources at any point in time. Stores output data from tasks in memory or on disk. It is important to note that workers and executors are aware only of the tasks allocated to them
    * [TaskRunner](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/Executor.scala#L317) is a thread of execution [created](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/Executor.scala#L240) when an executor is requested to [launch a task](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/Executor.scala#L239).
    [Creates](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/Executor.scala#L410) TaskMemoryManager 
      * [TaskMemoryManager](https://github.com/apache/spark/blob/master/core/src/main/java/org/apache/spark/memory/TaskMemoryManager.java) manages the memory allocated by an individual task
    
    * [ExecutorBackend](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/ExecutorBackend.scala) send updates to the cluster scheduler
      * [CoarseGrainedExecutorBackend](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/executor/CoarseGrainedExecutorBackend.scala) - an endpoint that uses a dedicated thread pool for delivering messages
         * [YarnCoarseGrainedExecutorBackend](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/executor/YarnCoarseGrainedExecutorBackend.scala) - implementation of CoarseGrainedExecutorBackend for YARN resource manager

      * [LocalSchedulerBackend](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/scheduler/local/LocalSchedulerBackend.scala) used when running a local version of Spark, sits behind a TaskScheduler and handles launching tasks on a single Executor
      
      
      
## Apache Spark execution strategies

**[Optimizer](https://github.com/apache/spark/blob/master/sql/catalyst/src/main/scala/org/apache/spark/sql/catalyst/optimizer/Optimizer.scala)** leverages advanced programming language features in a novel way to build an extensible query optimizer. 
On top of this framework, it has libraries specific to relational query processing (e.g., expressions, logical query plans), and several sets of rules that handle different phases of query execution: analysis, logical optimization, physical planning, and code generation to compile parts of queries to Java bytecode

**[Join selection](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L138)** 

  Selects the proper physical plan for join based on join strategy hints. 
  * [Matches a plan whose output should be small enough to be used in broadcast join](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L143)

**[StatefulAggregationStrategy](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L401)** 

   * Used to [plan](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L427) streaming aggregation queries that are computed incrementally as part of a [[StreamingQuery]]   

**[StreamingDeduplicationStrategy](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L441)**

  * Used to [plan](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L444) the streaming deduplicate operator

**[StreamingGlobalLimitStrategy](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L455)**

  * Used to [plan](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L480) the streaming global limit operator for streams in append mode

**[StreamingJoinStrategy](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L487)**

  * Used to [plan](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L494) streaming queries with Join logical operators
  
**[Aggregation](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L509)**

  * Used to [plan](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L534) the aggregate operator for expressions based on the AggregateFunction2 interface  

**[Window](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L571)**

  * Used to [execute](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L573) window functions

**[InMemoryScans](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L589)**
  
  * Used to [explain](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L591) strategy for in-mem tables
  
**[StreamingRelationStrategy](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L607)**

  * Used to [explain](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L608) Dataset/DataFrame. Won't affect the execution
  
**[FlatMapGroupsWithStateStrategy](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L623)**
  * Used to [convert](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/SparkStrategies.scala#L629) logical operator to physical operator in streaming plans  
  

References:

* https://static.googleusercontent.com/media/research.google.com/en/archive/mapreduce-osdi04.pdf

* http://people.csail.mit.edu/matei/papers/2010/hotcloud_spark.pdf

* http://people.csail.mit.edu/matei/papers/2012/nsdi_spark.pdf

* https://www.cse.ust.hk/~weiwa/teaching/Fall15-COMP6611B/reading_list/YARN.pdf

* https://groups.csail.mit.edu/tds/papers/Gilbert/Brewer2.pdf

* https://www.researchgate.net/figure/Overall-system-architecture-of-the-proposed-coordinated-cache-manager-in-Spark-Our_fig2_319327363