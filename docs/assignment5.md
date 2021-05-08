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

 
