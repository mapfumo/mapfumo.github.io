---
layout: post
title: Developing Data Products with Python
date: 2015-09-27 19:16:44
categories:
---

Python has great libraries (Scikit-learn, NumPy, Scipy, Pandas, Matplotlib and Seaborn for visualisation) for data science applications and application development in general. I took the famous machine learning <a href="https://en.wikipedia.org/wiki/Iris_flower_data_set">iris data set</a> and developed a <a href="http://ec2-52-26-183-208.us-west-2.compute.amazonaws.com">web based application</a> for determining an iris variety given sepal width, sepal length, petal width and petal length.

<h2>Deployment challenges</h2>
Initially I wanted to deploy the application on <a href="https://devcenter.heroku.com/articles/getting-started-with-python#introduction">Heroku</a> but getting Scipy and Numpy to work proved too cumbersome. I tried pre-built packs with Scipy but Matplotlib would not work with those. There is a nice <a href="https://github.com/kennethreitz/conda-buildpack">Heroku Buildpack for Conda pack</a> but that blew my application beyond the <a href="https://devcenter.heroku.com/articles/limits">300MB limit</a> imposed by Heroku.

<img src="/img/iris.png" alt="Initial Screen" style="width:804px;height:628px;">

Next I tried <a href="http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html">Amazon Elastic Beanstalk</a> and I ran into Matplotlib issues again. My application doesn't necessarily need to showcase the single graph in it to the user but visualisation is important for data science applications.

My final solution was to spin up an <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html">Amazon EC2 server</a> running Ubuntu Server 64-bit and install the required packages myself. Apparently the EC2 server I used comes with 1GB RAM and no swap file. While configuring Scipy, the Fortran compiler exhausted all the RAM. I had to configure a swap file first before finishing off. The adavantage this approach is that I got more explicit error messages that enabled me find solutions.

<img src="/img/iris_2.png" alt="Initial Screen" style="width:804px;height:628px;">
<hr>

<h2>Conclusion</h2>
Although this is a simple application its a launchpad for more advanced applications. And now that I have a good understanding of Flask I should have little problems in implementing future solutions using Django.

The <a href="http://ec2-52-26-183-208.us-west-2.compute.amazonaws.com">final application</a> can be found here on Amazon and the <a href="https://github.com/mapfumo/iris-flask-app">source here on GitHub.</a>
