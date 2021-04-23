In this assignment we analyze data provided by the city of Nijmegen by integrating two data sets - one with streetnames and their quarters and one with public artworks.

## Data Cleaning
We begin by using the BAG-file, which contains records of addresses and buildings. 

To load this file we use

```
val bagdata = spark.read.format("csv").option("header", true).load("file:///opt/hadoop/share/data/BAG_ADRES.csv").cache()
```
The dataset has fields like "ACTCODE", "ADRES_ID", "X_COORD" , "Y_COORD" and 16 more fields.

Then we create a dataframe of only 4 fields of the dataset using

```
val addrDF = bagdata.select('STRAAT, 'X_COORD, 'Y_COORD, 'WIJK_OMS)
```

We use .show() to show the rows of the dataframe, .describe().show() is used for showing the statistics of the value distributions. These are the statistics:
![image1](image1.png)

In these statistics we can see that the mean for X and Y coordinates is null which probably means that not all of the data has an X and Y coordinate. 
We can see how many X and Y coordinates by using 
```
addrDF.filter( $"X_COORD".isNull ).count
addrDF.filter( $"Y_COORD".isNull ).count
```
From this we find that there are 45 X_COORD and 45 Y_COORD missing. Since these records constitute a very small fraction of the total rcords (45 out of 100k), we can make the decision to leave these out.
To see the value of the STATUS field in the  records with the missing values we use
```
bagdata.filter($"X_COORD".isNull).show(45)
```
which shows that the status is "Niet authentiek".

With
```
bagdata.groupBy($"STATUS").count
```
we can see that this particular status only occurs in less than 1% of the data

