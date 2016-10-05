---
layout: page
title: Network Simulator 3 (NS-3)
sitemap:
    priority: 1.0
    changefreq: weekly
    lastmod: 2015-05-30T16:31:30+05:30
---
<link rel="stylesheet" href="css/monokai_sublime.css">
<script src="js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

# Network Simulator 3 (NS-3)

## ***Introduction***
One of the tools that I will be using for my research is <a
href="https://www.nsnam.org/">NS-3</a> (Network Simulator
Version 3). NS-3 is a discrete-event network simulator written in C++ with
Python bindings, targeted primarily for
research and educational use. 
Discrete event simulation models the behaviour of a complex system as an ordered sequence of
well-defined events. Time moves discretely from event to event.
This contrasts with continuous simulation in which the simulation continuously tracks the system dynamics over time.

## ***Installation on Ubuntu 16.04***
***Install prerequisites***
<pre><code class="bash">
sudo apt-get install gcc g++ python python-dev mercurial bzr gdb valgrind
gsl-bin libgsl0-dev libgsl2 flex bison tcpdump sqlite sqlite3 libsqlite3-dev
libxml2 libxml2-dev libgtk2.0-0 libgtk2.0-dev uncrustify doxygen graphviz
imagemagick texlive texlive-latex-extra texlive-generic-extra
texlive-generic-recommended texinfo dia texlive texlive-latex-extra
texlive-extra-utils texlive-generic-recommended texi2html python-pygraphviz
python-kiwi python-pygoocanvas libgoocanvas-dev python-pygccxml
</code></pre>

***Download NS-3 (current version is 3.26 as of this writing)***
<pre><code class="bash">
http://www.nsnam.org/release/ns-allinone-3.26.tar.bz2
tar xfj ns-allinone-3.26.tar.bz2
cd ns-allinone-3.26
</code></pre>

***Build, configure and test***
<pre><code class="bash">
./build.py --enable-examples --enable-tests
cd ns-3.26
./waf -d debug --enable-examples --enable-tests configure
./waf
./test.py
</code></pre>

### ***Configure Eclipse for NS-3 Coding***
1. Download Eclipse for C++ developers from <a
   href="http://www.eclipse.org/downloads/packages/eclipse-ide-cc-developers/mars2">here</a>. You also need Java.
2. Start **eclipse** and configure **mercurial**. You do this by going to
   ***Help*** -> ***Eclipse Marketplace...*** and search for ***"MercurialEclipse
   Project"***
3. Create a new C++ project (let's call its it ns326) in the ns-3 root folder. Select ***Cross GCC*** as
   the ToolChain {This is important}. Click ***Next***, leave "Debug" and "Release"
   selected for now and click ***Next*** and ***Finish***
4. Right click on your project (ns326 in my case), select ***Team*** -> ***Share
   Project...***. Select ***Mercurial*** then ***Next*** and ***Finish***
5. The next thing to do is to configure ***WAF***. Right click on the project
   (ns326 in my case) and choose properties. Select ***C/C++ Build*** and
   uncheck "Use default build command" and "Generate Makefiles automatically".
   Replace "Build command" with {your-project-path/waf}. In my case that would
   be ***/home/tony/ns-allinone-3.26/ns-3.26/waf*** and configure Build
   directory as {your-project-path/build},
   ***/home/tony/ns-allinone-3.26/ns-3.26/build*** in my case. Select
   ***Behavior*** tab and uncheck everything. Select ***Build (incremental
   build)*** again and type ***build*** then select ***Clean*** and ***Stop on
   first build error*** and click ***OK***
6. The last thing is to configure the debugger. Click ***Run*** -> ***Debug
   Configurations*** -> Under ***Project:*** click on browse and select your project (ns326 in my case),
   under ***C/C++ Application*** type and select ***scratch-simulator***.
   [Remove any spaces under project ***Name***. Select
   ***Environment tab*** and click ***New..*** variable and add ***LD_LIBRARY_PATH*** under name 
   and ***{Your project path}/build***, /home/tony/ns-allinone-3.26/ns-3.26/build in my case.
   Apply and close.
