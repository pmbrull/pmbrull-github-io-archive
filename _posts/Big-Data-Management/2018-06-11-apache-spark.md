---
layout: single
title: "Apache Spark"
date: 2018-06-11
categories: big-data-management
tags: bigdata management mapreduce depth map reduce sark shuffle
author_profile: false
mathjax: true
---

In this post we are going to talk about Apache Spark, starting by busting some myths or missconceptions:

* Spark tries to process *as much as possible* in memory, which does not mean that Spark uses in-memory processing. Things would just blow up considering it was born to be used in Big Data.
* It inherits MapReduce's **shuffle**, but tries to pipeline data transformations (chaining multiple maps and reduces) instead of writting each time to disk, which improves perfomance. However, data must go to disk some times in order to have a robust system against failure.
* Spark is just an evolved version of MapReduce, but you can achieve the same final results with one framework or the other (not talking about perfomance, which is already mentioned). What changes is that Spark allows for a more friendly environment and gives further information about transformations (so there can be internal optimizer mechanisms) instead of working with map and reduce as total black boxes.
* Spark DOES NOT DISTRIBUTE! It can't distribute processing further than what the underlying file system permits. It brings, however, the ability to easily distribute on a healthy cluster.

## From MapReduce to Spark

Following the common thread in these [series](https://pmbrull.github.io/big-data-management/), what needs got raised by MapReduce that Spark tries to tackle?

1. A framework capable of chaining MapReduce jobs in a more optimal way: Recall the synchronization barriers showed in last [post](https://pmbrull.github.io/big-data-management/map-reduce-II/). With an environment allowing pipelining of data transformations, we can avoid some of them.
2. Reuse operations with **data persistence**. In MapReduce, processes were independent from each other. With Spark we are trying to use previous operations to improve perfomance, for example to just read from disk once and run different tasks with that data as starting point, instead of accessing to disk multiple times.  
3. Reflect the friendlier environment also in work distribution and fault tolerance, raising the abstraction to the user for both tasks.
4. Unify an execution model with the introduction of RDDs (*Resilient Distributed Datasets*), which allow for different structures on them, like tables or graphs.
5. Improve the limited scope of the existing programming model by adding different APIs, ending up with three possible programming languages: Scala, Java and Python.
6. Improve resource sharing: in MapReduce resources are defined statically, while in Spark, the *Driver* notifies **YARN** a more accurate approximation of resources.

## Resilient Distributed Datasets

Getting into the name:

* Resilient: fault-tolerant and easy to recover failed partitions.
* Distributed: as data is located on multiple nodes.

Quoting Matei Zaharia, author of [Learning Spark](http://shop.oreilly.com/product/0636920028512.do) and one of the fathers of Spark, RDDs are

> Unified abstraction for cluster computing, consisting in a read-only, partitioned collection of records. They can only be created through deterministic operations on either (1) data in stable storage or (2) other RDDs.

This means that modifying an RDD ends up with another RDD: RDDs are **immutable** collections of data! This allows for easy data sharing among multiple processes and for caching intermediate operations to be used again without concerns, as concurrency is easier to achieve than when reading and writting onto the same collection of data with several jobs at the same time.

## Architecture
