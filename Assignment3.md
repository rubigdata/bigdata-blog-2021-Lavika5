Spark is a distributed processing system which is used for working on big data. It uses in-memory caching and optimized query execution for fast queries on data of any size. Resilient Distributed Datasets are important data structures used in Spark.  RDD is an immutable distributed collection of elements partitioned across the nodes of the cluster that can be operated on in parallel.

There are 2 ways of creating RDDs. In this assignment parallelize is used. This copies the elements of the collection so that they can be computed on in paralell. 

```
println( "Lines:\t", lines.count, "\n" + 
         "Chars:\t", lines.map(s => s.length).
                            reduce((v_i, v_j) => v_i + v_j))
```

```
println( "Lines:\t", lines.count, "\n" + 
         "Chars_longest:\t", lines.map(s => s.length).
                            reduce((v_i, v_j) => v_i max  v_j))
```


