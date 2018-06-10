---
layout: single
title: "Key-Value Stores"
date: 2018-06-09 16:00:00
categories: big-data-management
tags: bigdata management mapreduce key-value intro
author_profile: false
mathjax: true
---

In the last post we saw what Hadoop is and how HDFS works. Its goal is building a system that:

* Does not require an explicit schema, so one can freely insert all kind of files.
* Has an easy setup and can have easily evolve allowing the ingestion of bigger amounts of data: **Scalability**
* Has a short response time and small throughput (volume of information that flows accross the system and network).
* Is **reliable** and its data **available**, even in case of failure.

Before going any further, let's make a quick stop at **Key-Value** stores, eventhough they play a separate role than HDFS, its information structure will help us when diving into the internals of MapReduce.

## Key-Value Stores

Again, what needed to be solved so that new technology appears?

We have a **File System** with file chunks as atomicity. When accessing the information, the whole chunk has to be read, and the only operation possible is *append* (adding information at the end of the file, one can't *update* any information in the middle, so sometimes the only solution is a full re-write). 

> OBS: at the same time, when writting data, it is only materialized in the system when we have finished writting and the file is *closed*.

There is no problem when we are willing to access all data, but imagine retrieving 128MB of information just because you need to read a specific row. What we are looking at is a change of data querying paradigm, going from **Sequential** to **Random** access, so that the amount of useful information when reading is increased.

The data structure that allows improvement in these situations is *key-value*: 

![keyvalue]({{ "/assets/images/bigdatamanagement/keyvalue.png" | absolute_url }})

Where each key maps only to one value, query is performed on the key, and value is schemaless. In the example above, we may have information about an e-commerce users, and the field premium may not always be present.

A fine example of Key-Value Store would be **HBase**. I'll try to find to time to write about it another day!
