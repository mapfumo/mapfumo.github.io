---
layout: page
title: Scala Programming
sitemap:
    priority: 1.0
    changefreq: weekly
    lastmod: 2015-05-30T16:31:30+05:30
---
<link rel="stylesheet" href="css/monokai_sublime.css">
<script src="js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

# Scala
Scala is one of the languages I recommend one should have in their Data Science toolbox. To learn Scala a Java background would be ideal, other c/c++ family of languages would also be fine.
Scala  is a functional and object-oriented programming language based on Java and compiles to the JVM. It is one of the best languages for crunching big data. Apache Spark is written in Scala.

* Scala is statically typed although type information is not always necessary and it can be inferred.
* Scala is functional – In Scala functions are objects making them first class values
* Scala uses existing Java libraries and tools

<b>Declarations</b>

A variable can be declared with ```var```(can change later on) or ```val```(immutable)
<pre><code class="scala">
scala> var speed = 10 // we can change speed later
scala> val gravity = 10 //suitable for constants
scala> gravity = 20 // will generate an error, gravity is immutable
</code></pre>
<hr>
<b>Variable type Inference</b>

<pre><code class="scala">
scala> var x = 5 // x will be Int type
scala> x = "Antony Mapfumo" // This is illegal, error: type mismatch
scala> x = 20 // Fine, x is still an Int
scala> var x = "I am a string" // allowed, new declaration
</code></pre>
In the above example we could have explicitily declared "x" as an Int:- 
<pre><code class="scala">scala> var x: Int = 5</code></pre> 
It is one of the things I like about Scala as compared to Java, less typing.
<hr>
<b>Multiple assignment with type inference</b>

<pre><code class="scala">scala> val (a, b) = (5, "I am a string")</code></pre>

<b>"a"</b> will be assigned to Int: 5 whilst <b>"b"</b> will be String: "I am a string"

<hr>
<b>Function definition and the implied return</b>


Here is a simple function to return the sum of two numbers:-
<pre><code class="scala">scala> def addNums(x: Int, y: Int): Int = x + y</code></pre>

Notice how there is no return statement, again less typing. We can be implicit and include the return statement,

<pre><code>scala>def addNums(x: Int, y: Int): Int = { return x+y }</code></pre>
<hr>
## Resources
For an in-depth look at Scala I recommend the following resources:-

* <a href="https://www.coursera.org/course/reactive">Principles of Reactive Programming - Cousera</a>
* <a href="https://www.coursera.org/course/progfun">Functional Programming Principles in Scala - Cousera</a>

I found this video about Scala to be as educational as it is funny. Enjoy!

<iframe width="420" height="315" src="https://www.youtube.com/embed/LH75sJAR0hc" frameborder="0" allowfullscreen></iframe>

