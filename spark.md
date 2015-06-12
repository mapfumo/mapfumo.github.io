---
layout: page
title: Apache Spark
sitemap:
    priority: 1.0
    changefreq: weekly
    lastmod: 2015-05-30T16:31:30+05:30
---
<link rel="stylesheet" href="css/monokai_sublime.css">
<script src="js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

# Apache Spark

<a href='https://spark.apache.org/'>Apache Spark</a> is an open source big data processing framework built around speed and ease of use. Spark is written in Scala and runs on Java Virtual Machine (JVM). I am more interested in Spark for Machine Learning.

I have enrolled in <a href='https://www.edx.org/course/introduction-big-data-apache-spark-uc-berkeleyx-cs100-1x'>Introduction to Big Data with Apache Spark</a>. The course objectives include:-

* Learning how to use Apache Spark to perform data analysis
* How to use parallel programming to explore data sets
* Applying Log Mining, Textual Entity Recognition and Collaborative Filtering to real world data questions
* Preparing for the Spark Certified Developer exam

Having already completed the first laboratory assessment I can say that the course is very hands on and the exercises are very challenging. Although the exercises have to be completed in Python (PySpark) I am also completing them in Scala.

## PySpark
<pre><code class="html">
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.
Spark assembly has been built with Hive, including Datanucleus jars on classpath
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
15/06/12 15:20:32 WARN Utils: Service 'SparkUI' could not bind on port 4040. Attempting port 4041.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 1.1.0
      /_/

Using Python version 2.7.10 (default, May 28 2015 17:02:03)
SparkContext available as sc.

In [1]: wordsList = ['cat', 'elephant', 'rat', 'rat', 'cat']

In [2]: wordsRDD = sc.parallelize(wordsList, 4)

In [3]: wordsRDD.collect()
Out[3]: ['cat', 'elephant', 'rat', 'rat', 'cat']

In [4]: 
</code></pre>
<hr>
## Spark (Scala)

<pre><code>
Spark assembly has been built with Hive, including Datanucleus jars on classpath
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 1.1.0
      /_/

Using Scala version 2.10.4 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_45)
Type in expressions to have them evaluated.
Type :help for more information.
15/06/12 15:22:02 WARN Utils: Service 'SparkUI' could not bind on port 4040. Attempting port 4041.
15/06/12 15:22:02 WARN Utils: Service 'SparkUI' could not bind on port 4041. Attempting port 4042.
Spark context available as sc.

scala> val wordsList = Array("cat", "elephant", "rat", "rat", "cat")
wordsList: Array[String] = Array(cat, elephant, rat, rat, cat)

scala> val wordsRDD = sc.parallelize(wordsList)
wordsRDD: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[0] at parallelize at <console>:14

scala> wordsRDD.collect()
res0: Array[String] = Array(cat, elephant, rat, rat, cat)

scala>
</code></pre>
