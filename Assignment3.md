Spark is a distributed processing system which is used for working on big data. It uses in-memory caching and optimized query execution for fast queries on data of any size. Resilient Distributed Datasets are important data structures used in Spark.  RDD is an immutable distributed collection of elements partitioned across the nodes of the cluster that can be operated on in parallel.

There are 2 ways of creating RDDs. In this assignment parallelize is used. This copies the elements of the collection so that they can be computed on in paralell. 

When an RDD is created, a Directed Acyclic Graph is built and the graph is split into different tasks which are scheduled to execute at different stages.
```
println( "Lines:\t", lines.count, "\n" + 
         "Chars:\t", lines.map(s => s.length).
                            reduce((v_i, v_j) => v_i + v_j))
```
map is used as a transformation operation. It is used to return the same number of elements as input. Then we call the action reduce on the new RDD created using map. Because of lazy evaluation in Spark, the execution starts when an action is triggered.
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
For each input value "map" only produces one output whereas "flatMap" produces zero or more output values. In this case we are giving the lines as input and want words as ouput. We know that there is more than one word so we need more than one output value for an input value and so we use flatMap. 

We get multiple files because we have not defined the number of partitions for words so Spark automatically divides the result into 2 files - part-00000.crc and part-00001.crc.
 
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

Assignment 3b

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


