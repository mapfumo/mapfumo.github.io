---
layout: page
title: Linux tips and trips
sitemap:
    priority: 1.0
    changefreq: weekly
    lastmod: 2015-05-30T16:31:30+05:30
---
<link rel="stylesheet" href="css/monokai_sublime.css">
<script src="js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

# Linux tips and tricks
***Searching for files and files containing particular text***
<pre><code class="bash">
# find file(s) and delete
find . -iname "file-to-find" -exec rm -rf {} \;
# search for files containing particular text
grep -r[nw] "text-to-find" # n - print line number, w - match whole word
</code></pre>

### <a href="http://www.tecmint.com/20-funny-commands-of-linux-or-linux-is-fun-in-terminal/">20 Funny Linux Commands: Linux is Fun in Terminal</a>


