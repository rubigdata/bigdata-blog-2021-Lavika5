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
 
