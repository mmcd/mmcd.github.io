---
layout: post
title: "Getting Cassandra, Spark and Cuda to be interoperable"
date: 2017-05-22 20:54:22 EDT
category: [cluster, research]
tags: [cassandra, spark, gpgpu, cluster, research]
author: "michael"
---

This is fairly non-trivial to figure out, but in hindsight seems easy now.

The big things:
* I opted for Pyspark because python
* I opted for Pycuda because pycuda
* I opted for the cassandra driver in python because... see the pattern?

Leaning on python made everything a bit easier I think. I have gotten Hadoop and Cuda to talk to each other quite easily but its a chore getting all the nodes working together and making sure every node is using the right version of Cuda and Java and the JCuda packages. Python is a ton easier in that respect. Plus there is a lot of other little things that get brought to the table down the line that I want to look at such as machine learning, embedding graph databases in cassandra, etc. python should just make that a bunch easier. I am willing to take the performance hit for the interoperability.

# Pycuda notes
* Everything you pass to your kernel has to be some sort of numpy construct. EVERYTHING. I spent about an hour wondering why things were not working because I was passing a standard python int.

* using arrays of structs is both easy and a total pain, I still do not understand the whole argument between SOA and AOS. pycuda makes it almost painfully hard and obtuse to use arrays of structs compared to just passing multiple arrays of standard types.

# Pyspark notes
* You want to get the most out of your gpgpu kernels, so you want to dump as much on the gpu as possible in order to
maximisze the parallelism as you can. This means you somehow have to pass a chunk of a dataframe or rdd to the GPU at once. Thankfully if you can figure out how to get the right sized partition then this isn't much of a problem. You can solve this by changing the executor memory, sort of, in the spark configs.

* you can also manually break the partitions up. I didn't see much benefit to this though as it didn't really seem to do much for me,.

* you want to be really careful work multiple workers per node. gpus are resource starved and you can run out of memory if you arent careful

* I found foreachPartition() helpful here. it doesn't return anything itself and you cannot write back to the rdd since its immutable but since those weren't really problems for me... What was useful was writing directly inside that loop back to cassandra with batched writes. I wanted, at least at this stage to write back out to cassandra. It slows things down but also means I don't ever have to compute certain things ever again.

# Cassandra notes
* there isn't much to say here, you are going to grab things out of cassandra based on cassandra's partitions as dataframes. 


## Model overview

* Pyspark->Get partition from cassandra->foreach partition:
**(launch kernel->compute on partition->writeback to cassandra)