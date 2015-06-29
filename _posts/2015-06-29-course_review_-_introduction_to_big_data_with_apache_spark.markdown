---
layout: post
title: Course Review - Introduction to Big Data with Apache Spark
date: 2015-06-29 12:50:59
categories:
---

<a href="https://www.edx.org/course/introduction-big-data-apache-spark-uc-berkeleyx-cs100-1x">CS100.1x, Introduction to Big Data with Apache Spark</a>, is a five week-long course on working with Big Data using <a href="https://spark.apache.org/">Apache Spark</a>. I have just completed the last of four very engaging lab assignments using PySpark for data analysis.

<strong>Course prerequisites</strong>

A programming background and experience with Python is recommended. I would recommended a detailed understanding of Python list comprehension and lambda functions as these are all over the place in the lab assignments.

<strong>Lab assessments</strong>

I enjoyed the very practical practical nature of the course. Depending on your previous knowledge of Spark and Python each lab can take an average of 5-6 hours.

<strong>Lab 1 – Word count in all of Shakespeare's plays</strong>

The emphasis here is to learn and apply the Spark data model, transformations and actions. The lab shows how Spark can scale from analysing words in a simple sentence to a large collection like Shakespeare's plays.

<strong>Lab 2 – Web Server Log Analysis with Apache Spark</strong>

Building upon the foundations laid in lab 1 the goal here is analyse a massive collective of server log files, NASA Apache web server logs. The assessment shows how to use Apache Spark on real-world text-based production logs and fully harness the power of that data. Tasks include:-

* Configuration and Initial RDD Creation
* Data cleaning
* Content size statistics, frequent hosts, Response code analysis
* Response Code Graphing with matplotlib, visualising endpoints
* Top 10 error endpoints, number of unique hosts
* Visualizing the Average Daily Requests per Unique Host
* Exploring 404 Response Codes

<strong>Lab 3 - Text Analysis and Entity Resolution</strong>

Entity resolution is a common, yet difficult problem in data cleaning and integration. This lab will demonstrate how we can use Apache Spark to apply powerful and scalable text analysis techniques and perform entity resolution across two datasets of commercial products.

* Bag of words, stop words
* ER as Text Similarity - Weighted Bag-of-Words using TF-IDF (Term-Frequency/Inverse-Document-Frequency)
* Corpus creation
* ER as Text Similarity - Cosine Similarity


<strong>Lab 4 - Introduction to Machine Learning with Apache Spark</strong>
One of the most common uses of big data is to predict what users want. This allows Google to show you relevant ads, Amazon to recommend relevant products, and Netflix to recommend movies that you might like. This lab will demonstrate how we can use Apache Spark to recommend movies to a user. We will start with some basic techniques, and then use the Spark MLlib library's Alternating Least Squares method to make more sophisticated predictions.

* Predicting Movie Ratings from a subset dataset of 500,000 ratings
* Basic recommendations
* Collaborative filtering

<strong>Conclusion</strong>

I enjoyed the course very much and I have enrolled in the follow-up course, <a href="https://www.edx.org/course/scalable-machine-learning-uc-berkeleyx-cs190-1x">CS190.1x: Scalable Machine Learning.</a> This course has also given me the foundations I need for the <a href="http://go.databricks.com/spark-certified-developer">Apache Spark Developer Certification.</a>
