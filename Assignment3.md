# Assignment 3

# Setup 
I created a docker in linux to run Spark using 
```
docker create --name hey-spark -it -p 8080:8080 -p 9001:9001 -p 4040:4040 rubigdata/course:a3
```
Then I ran 
```
docker start hey-spark
```
After that I opened Zepplin in the browser and imported the notebooks for the assignment from the github repository. 

# Introduction to Spark
Spark is a distributed processing system which is used for working on big data. It uses in-memory caching and optimized query execution for fast queries on data of any size. Resilient Distributed Datasets are important data structures used in Spark.  RDD is an immutable distributed collection of elements partitioned across the nodes of the cluster that can be operated on in parallel.
There are 2 ways of creating RDDs. In this assignment parallelize is used. This copies the elements of the collection so that they can be computed on in paralell. 
Spark uses lazy evaluation which means that the transformations are lazy and when an operation in RDD lineage is called it does not execute immediately. Instead Spark keeps a record of the operations that are called using DAG(explained below). This has some advantages as it reduces the number of pass overs on the data increases speed. 
RDD persistence is an optimization technique that saves the results of the RDD evaluations and intermediate resuts for future use. This reduces overheads in computation. This can be done by using cache() method. This method stores the RDD in memory so that it can be used in parallel computations to increase efficiency. cache memory of Spark is fault tolerant so that if any partition of RDD is lost, it can be recovered by the orignal transformation operation. 

### Advantages of Spark
- Speed - Spark uses RAM instead of local memory for data storage
- Dynamic nature - computes in parallel
- low-latency data processing capability
- Supports graph analytics and machine learning

### Disadvantages of Spark
- cannot automatically optimize code 
- Does not have its own file management system and has to use other platforms like Hadoop
- Has a large number of small files instead of few large files
- Does not support multi-user enviornment

## Assignment 3a

### Why did Spark fire off eight different tasks?
When an RDD is created, a Directed Acyclic Graph(DAG) is built and the graph is split into different tasks which are scheduled to execute at different stages. So specifying 8 partitions  fires off 8 different tasks.

### Counting words
```
println( "Lines:\t", lines.count, "\n" + 
         "Chars:\t", lines.map(s => s.length).
                            reduce((v_i, v_j) => v_i + v_j))
```
map is used as a transformation operation. It is used to return the same number of elements as input. Then we call the action reduce on the new RDD created using map. 
The output was 
```
(Lines: ,147838,
Chars: ,5545144)
```
To find the longest line sentence I used the following code:
```
println( "Lines:\t", lines.count, "\n" + 
         "Chars_longest:\t", lines.map(s => s.length).
                            reduce((v_i, v_j) => v_i max  v_j))
```
It works in a similar way as counting the lines and characters in the corpus but instead of calculating the total number of characters in all the lines, I compared the number of characters in all the lines and displayed the maximum characters in any line.
The output was
```
(Lines: ,147838,
Chars_longest: ,78)
```
### Why did we use flatMap() instead of map()?
For each input value "map" only produces one output whereas "flatMap" produces zero or more output values. In this case we are giving the lines as input and want words as ouput. We know that there is more than one word so we need more than one output value for an input value and so we use flatMap. 

### Why are there multiple result files?
On running 
```
%sh 
ls -al /opt/hadoop/wc
```
The output is
```
total 8992
-rw-r--r-- 1 hadoop hadoop    ._SUCCESS.crc
-rw-r--r-- 1 hadoop hadoop    .part-00000.crc
-rw-r--r-- 1 hadoop hadoop    .part-00001.crc 
```
We get multiple files because we have not defined the number of partitions for words so Spark automatically divides the result into 2 files - part-00000.crc and part-00001.crc.

### Why is the count different?

When we use
```
val words = lines.flatMap(line => line.split(" "))
              .filter(_ != "")
              .map(word => (word,1))
val wc = word.reduceByKey(_ + _)
wc.filter((_._1 == "Macbeth").collect
```
The output shows that Macbeth apears in the text 30 times. Because we are just counting how many times the word "Macbeth" appears in the text.
But when we use  
```
val words = lines.flatMap(line => line.split(" "))
              .map(w => w.toLowerCase().replaceAll("(^[^a-z]+|[^a+z]+$)", ""))
              .filter(_ != "")
              .map(w => (w,1))
              .reduceByKey(_ + _) 
```
The output shows that it appears 285 times. This happens because
- we have converted all the letters to lower case so any occurences of "macbeth" are counted. 
- we have replaced all special characters with empty strings so any occurence of "Macbeth" with a special characters like ".", "!"  next to it will also be counted.

## Assignment 3b

### Partitions
A partition is a logical chunk of large distributed data set. It can be used to distribute work across clusters, dividing a task into smaller tasks and even in reducing memory requirements for each node.
When we submit a job into the cluster, each partition is further processes by specific executors. This is done one at a time so time taken is directly proportional to the size and number of partitions. So with more partitions it can take longer to complete the job because even though the size of the partition may reduce the number of partitions will increase, increasing the number of executors needed. But using a smaller number of partitions can be faster because even though the executor will have to  work on more data, the number of executors needed will decrease.

I ran  cat /proc/info in the docker shell and got the output of 2 processors. The output of the second one:
```
processor       : 1
vendor_id       : GenuineIntel
cpu family      : 6
model           : 142
model name      : Intel(R) Core(TM) i7-8565U CPU @ 1.80GHz
stepping        : 11
cpu MHz         : 1992.004
cache size      : 8192 KB
physical id     : 0
siblings        : 2
core id         : 1
cpu cores       : 2
apicid          : 1
initial apicid  : 1
fpu             : yes
fpu_exception   : yes
cpuid level     : 22
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single fsgsbase avx2 invpcid rdseed clflushopt md_clear flush_l1d arch_capabilities
bugs            : spectre_v1 spectre_v2 spec_store_bypass mds swapgs itlb_multihit srbds
bogomips        : 3984.00
clflush size    : 64
cache_alignment : 64
address sizes   : 39 bits physical, 48 bits virtual
```

The differences between the processors was their core id, apicid, initial apicid.


pairRDD is a transformation that arranges the data into two parts, with a Key as the first part and value as the second part.
rddPairs.partitioner is used to define the key which will be used to divide the elements. 
```
val rddPairsPart2 = rddPairs.partitionBy(new HashPartitioner(2))
```
HashPartitioner is used to define the number of partitions.
In this case there are 2 partitions. 

Then we group the pairs by their keys using the sequence x%100. The output is 
```
res30: String =
(8) ShuffledRDD[18] at groupByKey at <console>:29 []
 +-(8) MapPartitionsRDD[2] at map at <console>:29 []
    |  ParallelCollectionRDD[1] at parallelize at <console>:29 []
```

Running it again we get
```
(8) ShuffledRDD[19] at groupByKey at <console>:29 []
 +-(8) MapPartitionsRDD[2] at map at <console>:29 []
    |  ParallelCollectionRDD[1] at parallelize at <console>:29 []
```
This happens because groupByKey() depends on the way that the input is partitioned.

 
```
printf( "Number of partitions: %d\n", rddPGP2Count.partitions.length)
rddPGP2Count.partitioner
```
The output for the above segment is
```
Number of partitions: 2
res18: Option[org.apache.spark.Partitioner] = None
```
This is because when an RDD is made using map() the partitioner is not preserved.

The results are different for rrdA and rddB are different because rddA is transformed using map() while rddB is transformed using mapValues(). 
map() operates on the keys and values while mapValues() only operates on the values. This is the reason that the partitioner is not preserved when we use map() as it operates on the key as well and forgets the partitioner and reverts to the default partitioning. mapValues preserves the partitioner.

```
val rddC = rddA.repartition(2)
val rddD = rddB.coalesce(2)
```
The difference between the query plans of rddC and rddD is because repartition() does a full shuffle of the data and divides it equally into the partitions and can be used to increase or decrease the number of partitions. coalesce() does not do a full shuffle and instead uses HashPartitioner to adjust the data into the existing partitions and hence reduces the number of partitions.
That is the rease that the second quesry plan has one less shuffle phase. 
