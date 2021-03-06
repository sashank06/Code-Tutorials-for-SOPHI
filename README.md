# SOPHI Code

## Introduction

This github provides a basic tutorial on running PySpark on [SOPHI](http://sophi.uncc.edu).

## Step 1: Accessing SOPHI

To access SOPHI, you must have an active UNCC ID username (student, faculty or staff) and be connected to the UNCC network either directly (edu-roam) or through VPN. See this [link](https://faq.uncc.edu/pages/viewpage.action?pageId=6653379) on how to set up VPN access.

This link ([https://cci-hadoopm3.uncc.edu](https://cci-hadoopm3.uncc.edu)) provides access to SOPHI's Hue Interface.

To start, click this link and then when prompted, enter your UNCC ID and password.

## Step 2: Opening a Notebook

Within SOPHI, click the "Notebook" button on the top ribbon and click the "+ Notebook" button to create a new Notebook.

Once within a new Notebook, create a PySpark session.

## Step 3: Load Charlotte Sample Twitter Gnip Dataset

This step imports pySpark functions, reads in the Charlotte Gnip json file (Dec-Feb 2016 Geolocated Charlotte Tweets) and creates an RDD-file called tweets.

Last, the `printSchema()` function will print the schema so you can see the structure of the dataset.

```{python}
from pyspark.sql import SQLContext
from pyspark.sql.functions import substring

sqlContext = SQLContext(sc)
tweets = sqlContext.read.json("/twitter/gnip/public/charlotte.json")
tweets.printSchema()
```

## Step 4: Basic GroupBy & Count Functions

Next, let's explore two fields to groupby and count the tweets.

First, we groupby the `verb` field which corresponds to Tweets (post) and Retweets (share). Notice, this dataset only includes posts as the filtering rules for this dataset excluded Retweets.

Next, we groupby the `geo.type` field to identify whether the geolocated Tweets are points or places (NULL).

```{python}
#Retweets (share) vs Original Content Posts (post)
tweets.groupBy("verb").count().show()

#Geolocated Points vs non-Geolocated Points
tweets.groupBy("geo.type").count().show()
```

## Step 5: groupBy, filter and orderBy functions

```{python}
from pyspark.sql.functions import col

# by top 20 locations
tweets.groupBy("actor.location.displayName").count().orderBy(col("count").desc()).show()

# Tweets by users with more than 500k Followers Ordered by Tweet Time (postedTime)
tweets.filter(tweets['actor.followersCount'] > 500000).select("actor.preferredUsername","body","postedTime","actor.followersCount").orderBy("postedTime").show()
```

## Step 6: Create Spark DataFrame

Alternatively to creating RDD's, you can create a Spark DataFrame by using the `sqlContext.sql` function.

```{python}
from pyspark.sql import SQLContext
sqlContext = SQLContext(sc)
df = sqlContext.sql("SELECT id, postedTime, body, actor.id, actor.displayName, actor.location.displayName FROM tweets")
```

## Further Links

* [Spark SQL and DataFrames Tutorial](http://spark.apache.org/docs/latest/sql-programming-guide.html)

* [Spark Machine Learning Library Tutorial](http://spark.apache.org/docs/latest/ml-guide.html)