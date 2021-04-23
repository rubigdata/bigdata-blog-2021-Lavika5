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

###  Can we proceed without these records? Can you write the small numbers of lines of code to inspect these records?

To see the value of the STATUS field in the  records with the missing values we use
```
bagdata.filter($"X_COORD".isNull).show(45)
```
which shows that the status is "Niet authentiek".

With
```
bagdata.groupBy($"STATUS").count
```
we can see that this particular status only occurs in less than 1% of the data and with 
```
addrDF.filter($"X_COORD".isNull).show(45) 
```
we can see that this data contains addresses from different streets and quarters which seem to be random and do not have some clear connection to each other.

So in this particular instance we can delete these records as it should not have any significant impact on the results.

To provide a more structured interface to the data we define a case class with street and quarter as strings and X and Y coordinates as float values to represent the data using
```
case class Addr(street:String, quarter:String, x:Float, y:Float)
```
Then we project this case class into a dataset.

However when we select the X and Y coordinates as floats we get null value for all instances. This is because the coordinates contain commas instead of dots.
So to fix this we use  a function that converts the data to floats using Dutch language settings (which uses commas):
```
import java.text.NumberFormat
import java.util.Locale
val nf = NumberFormat.getInstance(Locale.forLanguageTag("nl")); // Handle floats written as 0,05 instead of 0.05

def convToFloat(s: String): Option[Float] = {
  try {
    Some(nf.parse(s).floatValue)
  } catch {
    case e: Exception => None
  }
}
import java.text.NumberFormat
import java.util.Locale
```
### Did we use the dataset or the dataframe API in the cell:
```
printf(
  "%d points at origin, %d null values",
  addrDF.filter("x = 0 or y = 0").count,
  addrDF.filter('x.isNull or 'y.isNull).count
)

45 points at origin, 0 null values
```
We used the dataset because with this new function values with missing coordinates give ‘0’ for the coordinates instead of ‘Null’. 
And this has an impact on the standard deviation and other values such as min because records that actually have null as the value for the coordinates will now have 0, while the other records will have much higher values.

To calculate the number of addresses for each quarter we use:
```
val qc_1 = addrDF.groupBy("quarter").count.cache()
```
Then we can calculate the quarters with the most addresses using:
```
val qc_1_top = qc_1.orderBy(desc("count")).limit(10)
```
The result for this is:
![image2](image2.png)

## SQL

Using SQL makes working on larger and more complicated queries easier.

We can register the DataFrame as SQL temporary view by using:
```
addrDF.createOrReplaceTempView("addresses")
```
In SQL to determine the largest quarters we use:
```
val qc_2_top = spark.sql("SELECT quarter, count(quarter) AS qc FROM addresses GROUP BY quarter ORDER BY qc DESC LIMIT 10")
```
By looking at the query plans of SQL and DataFrame operators, I could see that in the optimized logical plan, there were lesser steps taken in SQL. And in the physical plan, the order of the steps were different but they looked similar.

## Artworks

To use art data we load the file “kunstopstraat-kunstwerk.csv”using:
```
val kunst = spark.read
    .format("csv")
    .option("header", "true") // Use first line of all files as header
    .option("inferSchema", "true") // Automatically infer data types
    .load("file:///opt/hadoop/share/data/kunstopstraat-kunstwerk.csv").cache()
```
This file contains the fields: naam, bouwjaar, kunstenaar, locatie, latitude, longitude, omschrijving, eigendom, bron en url.

To get a sample of the data we use:
``` 
kunstwerken.sample(true,0.1).show()
```
The result of which is
![image3](image3.png)

When we analyze the sample of the data we can see that all records have a name, but a lot of the other fiels are filled in incorrectly or ‘null’.
When the ‘bouwjaar’ is not a number then the other fields of that record are often ‘null’. 
We can  also see that the latitude and longitude are different from the coordinates of the bagdata.
From the records with bouwjaar below 2000 only one doesn’t have a location. This record can be found using:
```
spark.sql("SELECT * FROM kunstxy WHERE locatie IS NULL").show().
```

