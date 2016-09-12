---
layout: post
title: Scala Tuples
date: 2015-09-03 19:41:26
categories: Scala
---
Scala tuples combine a fixed number of ***immutable*** items (uniform or mixed) together so they can be passed around collectively.

* Tuples are handy when you need to return multiple objects from a function.
* Together with Scala Case Classes (more on that in a future blog) they are essential when working with <a href='/spark.html'>Spark RDDs.</a>
<link rel="stylesheet" href="/css/monokai_sublime.css">
<script src="/js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

<pre><code class="scala">
scala> val intTuple = (11, 10, 8)
intTuple: (Int, Int, Int) = (11,10,8)

scala> // tuples are immutable, lets try to declare a mutable tuple
scala> var doubleTuple = (1.0, 8.9)
doubleTuple: (Double, Double) = (1.0,8.9)

scala> // lets change a value, should get an error message
scala> doubleTuple._1 = 4.0
console:8: error: reassignment to val
       doubleTuple._1 = 4.0
                      ^
scala> // tuples contain mixed items

scala> val shapeTuple = ("Circle", 5, true)
shapeTuple: (String, Int, Boolean) = (Circle,5,true)
</code></pre>

## Accessing items in a tuple
Items in a tuple can be accessed using the underscore syntax. The first element is **_1**, second is **_2**, and third is **_3**, and so on.
<pre><code class="scala">
scala> intTuple._1
res0: Int = 11

scala> shapeTuple._3
res2: Boolean = true
</code></pre>

## Returning multiple values from a function
Suppose we want the circumference and area of circle:-
<pre><code class="scala">
scala> def areaCircumference(radius: Int) = (math.Pi*radius*radius, 2*math.Pi*radius)
areaCircumference: (radius: Int)(Double, Double)

scala> val (area, circumference) = areaCircumference(5)
area: Double = 78.53981633974483
circumference: Double = 31.41592653589793
</code></pre>

## Tuples in Spark RDD
I have a large a collection of circle radii and I want to calculate the areas and cicurmference. We have already created a function that returns the area and radius as a tuple.
<pre><code class="scala">
scala> val circleRadii = Array(2,5,45,6,7,3,5,8,55,9,11,76,71)
circleRadii: Array[Int] = Array(2, 5, 45, 6, 7, 3, 5, 8, 55, 9, 11, 76, 71)

scala> val rdd = sc.parallelize(circleRadii)
scala> rdd.map(x=>areaCircumference(x)).take(2)
res5: Array[(Double, Double)] = Array((12.566370614359172,12.566370614359172), (78.53981633974483,31.41592653589793))

scala> // lets get the areas only
scala> rdd.map(x=>areaCircumference(x)).map(x=>x._1).take(2)
res6: Array[Double] = Array(12.566370614359172, 78.53981633974483)
</code></pre>

## Tuples for pattern matching
<pre><code class="scala">
scala> def tupleMatch(tuple: Any ){
     | tuple match {
     | case (anInt: Int, aString: String) => println("I have an Integer and a String: "+ anInt + ", "+ aString)
     | case (x: Double, y: Double) => println(s"I have two doubles, $x and $y")
     | case ("Antony Mapfumo", _, _) => println("Found Antony Mapfumo")
     | case _ => println("Unknown")
     | }
     | }
tupleMatch: (tuple: Any)Unit

scala> tupleMatch(3.4, 5.9)
I have two doubles, 3.4 and 5.9

scala> tupleMatch("Antony Mapfumo", 1, "This")
Found Antony Mapfumo

scala> tupleMatch(3, "Hello")
I have an Integer and a String: 3, Hello
</code></pre>

## Scala Tuples using shapeless
If you want tuples to behave like other Scala collections its better to use the <a href="https://github.com/milessabin/shapeless/wiki/Feature-overview:-shapeless-2.0.0#hlist-style-operations-on-standard-scala-tuples">Shapeless library</a>
