---
layout: single
title: "Jupyter Pyspark"
date: 2018-07-26
categories: spark-series
tags: spark python jupyter cheatsheet
author_profile: false
mathjax: true
toc: true
---

I was trying to get everything set up for working in the Master's final project. It was a while since I last used python with Spark, and when I did, I simply used the godly

```bash
pip install pyspark
```

to play around a bit. However, now I needed everything to run smoothly on the local architecture I had previously set up, so here it is just a small summary on how I could finally use my local cluster connected to a Jupyter Notebook thanks to this [link](https://blog.sicara.com/get-started-pyspark-jupyter-guide-tutorial-ae2fe84f594f).

* First of all, make sure that you have correctly installed Spark and set up the proper paths in your `~/.bashrc` file:

```bash
export SPARK_HOME=/opt/spark
export PATH=$SPARK_HOME/bin:$PATH
```

* Check that you can run `pyspark` command to start the API shell after re-opening the terminal or using `source ~/.bashrc`.
* Install Jupyter on your environment and update PySpark driver variables:

```bash
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='notebook'
```

Refresh the terminal again and by executing `pyspark`, Jupyter's UI should appear. 

However, note that you do not need to initialize the SparkContext again (actually, it will fail), as one context already was launched by the PySparkShell.

 ![png]({{ "/assets/images/sparkseries/spark-with-jupyter.png" | absolute_url }})

Another cool thing to learn from the mentioned post is the package `findspark` (*pip install findspark*), which lets you easily import Spark in any IDE:

```python
import findspark
findspark.init()

import pyspark
```

Hope this was useful to someone ;)