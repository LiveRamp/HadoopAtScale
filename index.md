# How to scale Hadoop to 100,000 jobs a day

LiveRamp runs Hadoop at scale -- our Hadoop environment runs around 100,000 CPU cores, 300TB of memory and 100PB of raw storage across 2,500 servers. 

We didn’t start at scale though; our Hadoop cluster started with 40 (small) servers.  As the business grew, so did our Hadoop environment.  With each order of magnitude increase (10 servers, 100 servers, 1000 servers), processes that once were easy to handle manually once in a while became a huge maintenance burden.

Scaling the Hadoop environment is half of the story; we also scaled the number of engineers writing big data applications, and likewise the number of unique applications.  This brought just as many challenges:

* If a team of 5-10 engineers are developing 10-20 applications, a couple infrastructure engineers know all of the important applications, and can attribute global performance problems to a known “bad apple” job.
* If a team of 50-60 engineers are running 100+ applications, no team can be expected to understand what every job is doing.  Only truly generic application monitoring can separate “normal” from “pathological” applications.

This article will walk through a number of the common data-processing problems Hadoop encounters at scale, and explain:

- **Why**: What is breaking
- **Symptoms**: The symptoms of the problem
- **Identifying and monitoring**: How to both reactively and proactively identify bad application patterns
- **Fixes**: How can we re-engineer applications to not cause these problems

A key refrain in this post will be, once you identify a usage pattern which is causing performance problems across your environment, _set up monitoring to make it easy to identify in the future_. Problems which make things 5% slower are rarely RCA’d (if they are even identified); only once the environment is so degraded that SLAs are missed will action be taken.  But 5% performance losses across a large environment is a significant cost driver.

The initial articles in this repository focus on problems encountered in LiveRamp's specific environment.  Specifically, this means

- Most poor performing applications are MapReduce jobs
- Cloudera Manager is available to view metrics
- Worker nodes use an array of HDD volumes as shared NodeManager and HDFS storage

The solutions outlined here will describe how to identify and fix problems in this environment, but are the issues are equally relevant in Ambari, EMR or Dataproc environments.

This repository is intended to be a living document; contributions from outside LiveRamp are welcome.

## Topics

[HDFS scaling: small files, or too many files](small_files.md)