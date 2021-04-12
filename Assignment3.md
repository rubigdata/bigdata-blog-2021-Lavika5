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

