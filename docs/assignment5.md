# Assignment5

In this assignment we are using Spark Structured Streaming to analyze the data from an online marketplace that sells RuneScape game items.

We are using HDFS for this assignment and to get rid of any errors that may occur we run 
```
%sh
sed -i -e 's|hdfs://localhost:9000|file:///tmp|' /opt/hadoop/etc/hadoop/core-site.xml
```

We start a simulation that generates a stream of events inside the docker container. The RuneScape like output is written to port 9999 by the simulator which is a python program. To do this we need to copy the program to the container and then execute it when we start the container.

Then we do a few imports so that we can work with the streams. Then we create dataframe and tie it to the TCP/IP stream n localhost port 9999 using
```
val socketDF = spark.readStream
  .format("socket")
  .option("host", "0.0.0.0")
  .option("port", 9999)
  .load()
```

Now we can start the data processing. I encountered alot of errors when I tried this but restarting the containers and deleting then importing the notebook again resolved these. Our goal at this point is to write data to the in-memory dataframe from the TCP/IP connection that we are listening on port 9999. We read all data that comes from the socket. I first tried this with a window of 1 second but that did not show any data. When I tried with a 5 second window I got 3 rows of data so then I ran it for 10 seconds adn got a good amount of data. Since there is alot of data and can cause the docker container memory to fill up we stop it.
```
val memoryQuery = streamWriterMem  
  .queryName("memoryDF")
  .start()

// Run for 1 second...
memoryQuery
  .awaitTermination(10000)
  
// ... and stop the query, to avoid filling up memory:
memoryQuery
  .stop()
```

memoryQuery is StreamingQuery that splits data into separate lines based on newline. To read this saved output and analyze it we use sql
```
select * from memoryDF LIMIT 10
```
The output of which is the first ten rows of the data 
![image1](9.png)

To count how many rows we got from the stream we use
```
spark.sql("select count(*) from memoryDF").show()
```
The output of which is 860 rows. 

## Parsing the data

Until now we only  read the data line by line without any structure. Once we figure out the structure we can create objects out of this data. To do so, we transform the data from String to a structure using a regular expression on the data from the stream before it is written to memory. I used:
```
val myregex = "\"^([A-Z].+) [A-Z].+ was sold for (\\\\d+)\""
val q = f"select regexp_extract(value, $myregex%s, 1) as material, cast(regexp_extract(value, $myregex%s, 2) as Integer) as price from memoryDF"
spark.sql(q).as[RuneData].show(10, false)
```
The output of which is:
![image2](11.png)

## Stream Processing

Until now we were reading the data and then processing it. But it would be more efficient and practical to process and read the data simultaneously using continous stream processing.
```
val consoleQuery = socketDF
  .writeStream
  .outputMode("append")
  .format("console")
  .start()
```
The above command continuously runs the  stream and instead of saving it to memory it writes it to the console in groups of 10 lines until it is stopped.

## Structured Console output

Now we structure the console output by using the same code as above. To do this we use class
```
RuneData with the attributes (material: String, tpe: String, price: Integer)
```
where tpe is the type.

The output of using the same regular expression here is:
![image3](12.png)

## Saving the stream data

We need to save the data from teh stream. To do this we create a folder to save the data. Then we need to read the data and write it to this folder.
```
val streamWriterDisk = runes
  .writeStream
  .outputMode("append")
  .option("checkpointLocation", "file:///tmp/checkpoint")
  .trigger(Trigger.ProcessingTime("2 seconds"))

val stream2disk = streamWriterDisk
  .start("file:///opt/hadoop/share/runedata")
```
Then we need to check if the stream is active for which we use 
```
spark.streams.active
```
As the program keeps running, it fills up the memeory. To check how much memory is used we can use
```
du --si /opt/hadoop/share/runedata
```

## Working with the collected data

We need to now analyze the collected data. To do this, first we run
```
val runes = spark
  .read
  .parquet("file:///opt/hadoop/share/runedata/part-*")
  .createOrReplaceTempView("runes")
```

We find the average price of the items of different materials grouped by their material and sorted in ascending order of the average price.
```
SELECT material, avg(price) FROM runes GROUP BY material ORDER BY avg(price)
```

![image4](22.png)

We can also analyze this graphically.

![image5](23.png)

It is very clear from the above graph and table that the average price of the items made of dragon material is much higher than the other materials. This is because the max price of dragon material is very high so it has a signiicant increase in the average.

### How many rune items were sold?

We can use the sql query 
```
SELECT COUNT(material) FROM runes WHERE material = "Rune"
```
The output of which is
### 581

### How many of each item type was sold?

We can use the sql query
```
SELECT tpe, COUNT(tpe) FROM runes GROUP BY tpe 
```
The ouput of which is 

![image6](24.png)

### How much gold was spent buying swords?

To do this I found the sum of the prices of all the items of that contained the word sword in the type.
```
SELECT tpe, SUM(price) FROM runes WHERE tpe LIKE '%Sword%' OR tpe LIKE '%sword%' GROUP BY tpe
```
The output of which is 
![image7](25.png)


