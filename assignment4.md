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

