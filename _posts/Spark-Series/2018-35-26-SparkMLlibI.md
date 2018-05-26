---
layout: single
title: "Spark MLlib (Part I)"
date: 2018-03-23
categories: machine-learning
tags: machine-learning spark spark-mllib
author_profile: false
mathjax: true
toc: true
---

Spark MLlib offers functions helping us implement Machine Learning algorithms. However, these algorithms require to have a solution or aproximation when being applied in parallel in a cluster, i.e they need tu support horizontal scaling. A fine example of that would be Distributed Random Forests. We will use versions greater than 2.0, from which the API started to be based on Datasets/Dataframes and not just RDDs. The main difference here is that, while both RDDs and Datasets/Dataframes are immutable distributed collection of elements of your data, Datasets/Dataframes have data organized as a tabular schema with structure defined when created (names and types).

And what about Dataframes and Datasets?

In Python API we don't have such a problem, as one can only work with DataFrames, where a DF is just a Dataset and each row is accessed by the generic type **Row**. Same definition applies to Java, though there exists a separated Dataset object, where one needs to define a row container.