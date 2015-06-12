---
layout: page
title: Data Visualisation
sitemap:
    priority: 1.0
    changefreq: weekly
    lastmod: 2015-06-10T16:31:30+05:30
---
<link rel="stylesheet" href="css/monokai_sublime.css">
<script src="js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

# Data Visualisation
 We are better at grasping the meaning of data when it is displayed visually (charts,
 graphs, heatmaps, etc) instead of tables, spreadsheets and raw text.
 Visualisation helps us to extract information that may not be initially
 obvious. A classic example of the importance of visualisation can be
 illustrated by <a href="http://en.wikipedia.org/wiki/Anscombe%27s_quartet">Anscombe's quartet</a>

### Illustrating the importance of data visualisation using Anscombe's quartet
<pre><code class="r">
> anscombe
   x1 x2 x3 x4    y1   y2    y3    y4
1  10 10 10  8  8.04 9.14  7.46  6.58
2   8  8  8  8  6.95 8.14  6.77  5.76
3  13 13 13  8  7.58 8.74 12.74  7.71
4   9  9  9  8  8.81 8.77  7.11  8.84
5  11 11 11  8  8.33 9.26  7.81  8.47
6  14 14 14  8  9.96 8.10  8.84  7.04
7   6  6  6  8  7.24 6.13  6.08  5.25
8   4  4  4 19  4.26 3.10  5.39 12.50
9  12 12 12  8 10.84 9.13  8.15  5.56
10  7  7  7  8  4.82 7.26  6.42  7.91
11  5  5  5  8  5.68 4.74  5.73  6.89

> summary(anscombe)
       x1             x2             x3             x4    
 Min.   : 4.0   Min.   : 4.0   Min.   : 4.0   Min.   : 8  
 1st Qu.: 6.5   1st Qu.: 6.5   1st Qu.: 6.5   1st Qu.: 8  
 Median : 9.0   Median : 9.0   Median : 9.0   Median : 8  
 Mean   : 9.0   Mean   : 9.0   Mean   : 9.0   Mean   : 9  
 3rd Qu.:11.5   3rd Qu.:11.5   3rd Qu.:11.5   3rd Qu.: 8  
 Max.   :14.0   Max.   :14.0   Max.   :14.0   Max.   :19  
       y1               y2              y3              y4        
 Min.   : 4.260   Min.   :3.100   Min.   : 5.39   Min.   : 5.250  
 1st Qu.: 6.315   1st Qu.:6.695   1st Qu.: 6.25   1st Qu.: 6.170  
 Median : 7.580   Median :8.140   Median : 7.11   Median : 7.040  
 Mean   : 7.501   Mean   :7.501   Mean   : 7.50   Mean   : 7.501  
 3rd Qu.: 8.570   3rd Qu.:8.950   3rd Qu.: 7.98   3rd Qu.: 8.190  
 Max.   :10.840   Max.   :9.260   Max.   :12.74   Max.   :12.500  

</code></pre>
From the data we can see:-

* identical means (9.0) for "x" variables, identical (7.5) means for "variables"
* identical variances, 11 for "x" and 4.1 for "y"

Now lets plot the data:-
<pre><code class="r">
library(ggplot2)
anscombe_long <- data.frame()
for(i in 1:4) {
  anscombe_long<-rbind(anscombe_long, data.frame(set=i, x=anscombe[,i], y=anscombe[,i+4]))
}
ggplot(anscombe_long, aes(x, y)) + geom_point(size=4,fill="steelblue", shape=21) + 
      geom_smooth(method="lm", fill=NA, fullrange=TRUE) + 
      facet_wrap(~set, ncol=2)
</code></pre>
<p><img src="/img/anscombe.png" alt="Anscombe Plots"></p>

Summary Statistics don’t give you a whole story about the data you are dealing with. Even though the regression lines above are pretty much the same, the plots clearly show how different the data is.
