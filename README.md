# pmbrull

To download PySpark for Windows in under 5min follow this [link](https://medium.com/@GalarnykMichael/install-spark-on-windows-pyspark-4498a5d8d66c).

**Getting Ready with GraphFrames on Windows**

Ok so, this has been quite a journey, and please disregard the *Quick* in Quick-Start as it's taken a few hours just to get the copy-paste example to work. Thus, I'm gonna write some steps down before I forget.

While looking through github issues and StackOverflow, people were asking for separate problems, but here i present you the complete saga, also with one unfound error that personally I don't understand, but hey, it works.

<div class="alert alert-info">
Note: These are the issues found when installing and running graphframes in Windows. It is common knowledge that windows cmd is lacking when compared to Unix. To being able to use Unix commands, install GOW for Windows.
</div>

1. First of all, download Spark from this [link](https://spark.apache.org/downloads.html). I just got the fresh Spark 2.3. I have it placed inside ```C:\```, so I will be using it as path example. Remember to set all options and paths to be able to use Pyspark with Jupyter.
2. Now, download **graphframes** package from the Spark packages [site](https://spark-packages.org/package/graphframes/graphframes). 
3. Unzip it and check that you got the directory: ```\graphframes\python\graphframes```. Inside you should have the following structure:
```
graphframes
│   __init__.py
│   graphframe.py
│   tests.py 
│   
└─── examples
│ 
└─── lib
```
Copy this inside your the Spark directory: ```C:\Spark\spark-2.3.0-bin-hadoop2.7\python\pyspark```, where other modules like MLlib and Streaming are placed.

4. Once you are set, run: ```pyspark --packages graphframes:graphframes:0.5.0-spark2.1-s_2.11```. You should see something like this
```
        ---------------------------------------------------------------------
        |                  |            modules            ||   artifacts   |
        |       conf       | number| search|dwnlded|evicted|| number|dwnlded|
        ---------------------------------------------------------------------
        |      default     |   5   |   0   |   0   |   0   ||   5   |   0   |
        ---------------------------------------------------------------------
```
However, here is where I found the first weird error. Spark could not locate nor download ```org.slf4j#slf4j-api;1.7.7 from local-m2-cache```. A fix for this is to download ```slf4j-api-1.7.7``` and place it under ``` C:\Users\Usuario\.m2\repository\org\slf4j\slf4j-api\1.7.7 ```. Please note that it is looking for version 1.7.7. I tried with newer ones but same problem appeared.

5. With these steps now you should be able to **import graphframes** modules. Nevertheless, fun goes on. Maybe I broke something when preparing the new Spark version, but I got a metastore_db and hive related error when creating the vertices and edges DataFrames. So let's give some attention to Hadoop now:

    1. Download Apache Hadoop. I am using hadoop-2.7.1. Place it under ```C:\hadoop```.
    2. Set ```HADOOP_HOME=C:\hadoop``` and include ```%HADOOP_HOME%\bin``` to the PATH environment.
    3. Create ```C:\tmp\hive ``` directory and run: ```winutils.exe chmod -R 777 \tmp\hive ```.

6. Remember how we downloaded slf4j-api version 1.7.7? Well, now you should be able to create both DataFrames, but when creating the GraphFrame, it is going to fail, as it does not find logging classes related to: ```scala-logging-slf4j_2.10-2.1.2.jar```. Yes, version 2.1.2. Download both ```scala-logging-slf4j_2.10-2.1.2.jar``` and ```scala-logging-api_2.10-2.1.2.jar``` and place them under the jars directory inside Spark.

Finally now, you should be good to go with the *NotSoQuick-Start Guide*. Hope this helped to save you some time and trouble!

