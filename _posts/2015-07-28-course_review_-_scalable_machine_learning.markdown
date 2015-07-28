---
layout: post
title: "Course Review - Scalable Machine Learning"
date: 2015-07-28 17:44:03
categories:
---

<a href="https://www.edx.org/course/scalable-machine-learning-uc-berkeleyx-cs190-1x">CS190.1x Scalable Machine Learning</a>, is a 5-week distributed machine learning course offered by UC Berkeley. This course is freely available on the edX MOOC (Massive Open Online Courses) platform. It is a follow up to another UC Berkely course, Introduction to Big Data with Apache Spark. I have just completed the last of the five laboratory exercises, Principal Component Analysis (PCA) using PySpark.

<strong>Course prerequisites</strong>
A programming background and experience with Python is recommended. I would also recommend first completing the course, <a href="https://www.edx.org/course/introduction-big-data-apache-spark-uc-berkeleyx-cs100-1x">Introduction to Big Data with Apache Spark</a>

<strong>Lab assessments</strong>

There is a nice selection of labs applicable in real life. Each lab is based on a lengthy iPython notebook consisting of several sections leading you through the process of implementing a machine learning algorithm with PySpark.

<strong>Lab 1 – NumPy, Linear Algebra, and Lambda Function Review</strong>

Gain hands on experience using Python's scientific computing library to manipulate matrices and vectors, and learn about lambda functions which will be used throughout the course.

* Math review
* NumPy
* Additional NumPy and Spark linear algebra
* Python lambda expressions

<strong>Lab 2 - Introduction to Apache Spark</strong>

Spark's Resilient Distributed Datasets (RDDs), data model, transformations, and actions, and write a word counting program to count the words in all of Shakespeare's plays.

<strong>Lab 3 - Linear Regression and Distributed Machine Learning Principles</strong>

The goal is to train a linear regression model to predict the release year of a song given a set of audio features. This lab covers a common supervised learning pipeline, using a subset of the Million Song Dataset from the UCI Machine Learning Repository

<strong>Lab 4 - Logistic Regression and Click-through Rate Prediction</strong>

Construct a logistic regression pipeline to predict click-through rate using data from a recent Kaggle competition. You will extract numerical features from the raw categorical data using one-hot-encoding, reduce the dimensionality of these features via hashing, train logistic regression models using mllib, tune hyper-parameter via  grid search, and interpret probabilistic predictions via a ROC plot.

* One-hot-encoding
* Sparse vectors
* Feature frequency visualisation
* Reduce feature dimension via feature hashing

<strong>Lab 5 - Neuroimaging Analysis via PCA</strong>

This lab delves into exploratory analysis of neuroscience data, specifically using principal component analysis (PCA) and feature-based aggregation. We will used a dataset of light-sheet imaging recorded by the Ahrens Lab at Janelia Research Campus, and hosted on the CodeNeuro data repository

<strong>Conclusion</strong>

The lecturers are very helpful and are very involved in the discussion forum. I enjoyed the course especially the last section, Principal Component Analysis and Neuroimaging. I am now doing the labs in Scala as well.



