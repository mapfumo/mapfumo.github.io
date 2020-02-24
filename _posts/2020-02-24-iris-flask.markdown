---
layout: post
title: Machine Learning web application using Python, Scikit-Learn, Flask and deployment on Heroku
date: 2020-02-21 17:41:28
categories: python
tags: [python, machine-learning]
---

The scikit-learn Iris data-set consists of 3 (Setosa, Versicolour, and Virginica) species (50 samples per species, for a total of 150 samples) of the iris flower. Each sample has four measurements: sepal length, sepal width, petal length, petal width. Given these measurements a machine learning model can predict the iris specie with a high degree of accuracy. Here I demonstrate a machine learning web application using *Python*, *Scikit-Learn* machine learning library and *Flask* web framework. The application is then deployed on [Heroku](https://iris-flask.herokuapp.com/) and the source code is on [GitHub](https://github.com/mapfumo/iris-flask). 

![](/img/iris.png)

Create a Python virtual environment and a dependencies file (important for [Heroku](https://www.heroku.com/) deployment) for the application. I have used [Anaconda](https://www.anaconda.com/) on Ubuntu 18.04.3

<pre><code class="bash">
# Create and activate virtual environment
conda create -n iris-flask python=3.6.10
conda activate iris-flask
# create list of required Python packages
pip freeze > requirements.txt
</code></pre>

#### **Deploy to Heroku**

*[Heroku](https://www.heroku.com/)* is a platform as a service (PaaS) that enables developers to build, run, and operate applications entirely in the cloud. Heroku offers a limited free plan. Head over there, create an account and install Heroku CLI client. Detailed instructions are [here](https://devcenter.heroku.com/articles/getting-started-with-python#set-up).

<pre><code class="bash">
sudo snap install heroku --classic
heroku login
heroku create iris-flask
# https://iris-flask.herokuapp.com is created
# list your apps.
heroku apps
</code></pre>

Install *[Gunicorn](https://gunicorn.org/)* web server and create a *Procfile*. 

<pre><code class="bash">
pip install gunicorn
# Procfile contents. run.py is the filename and iris_app is the app name
# iris_app = Flask(__name__)
web: gunicorn run:iris_app
# to upload we use Git
git init .
git add .
git commit -m "first commit"
</code></pre>

The app will be hosted at [https://iris-flask.herokuapp.com](https://iris-flask.herokuapp.com) 

[Interesting facts about irises](http://justfunfacts.com/interesting-facts-about-irises/)

