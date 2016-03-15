---
layout: post
title: "Scala and Spark at Workday"
description: ""
category: "spark"
tags: [spark, scala, bigdata]
---
{% include JB/setup %}

Scala is a big deal here at Workday. Like many companies we have a large Java codebase  developed over many years. Scala’s ability to seamlessly interoperate with Java code makes it easy to use Scala to build new functionality that interfaces with an existing codebase. Scala’s performance, rich type system, and emphasis on functional abstractions make it particularly well suited to big data applications.

The Workday Syman team focuses on building products to help customers make sense of all the data that they have stored in the Workday system. The name Syman comes from the data normalization technology that some of us built at our startup, Identified which was acquired by Workday in 2014. There were two previous iterations of the data pipeline; the first was written in Java, and the second in Scala. Neither of these were nearly as efficient or elegant as the one we have now, and that is largely due to another core technology, Spark.

![Hacking away on the data pipeline](/assets/scala-spark-workday/hacking_pipeline.jpg "Hacking away on the data pipeline")

Spark is one of the hottest technologies in big data today. Compared to Hadoop Map-Reduce, and it’s associated libraries, Spark is quite young, but in our year of using it, we have found it to be quite stable and mature. Part of this is due to the fact that Spark’s RDD is very close to Scala’s Collection API. Mirroring Scala’s Collections API is part of what makes Spark so easy to use, even without getting to some of Spark’s higher level APIs like Spark SQL and the ML pipelines API. I’ve been quite impressed at the rate of development that the Spark core team has maintained. Even very new features have been quite stable for us so far, and a couple of them are becoming core to our products.

One of the new Spark APIs that we have started using to great success is Spark Streaming. Like many streaming applications, both batch and streaming data processing are extremely important for our search indexing. This is part of what makes Spark Streaming well suited to search indexing. As data is updated, the streaming job takes care of keeping indexes up to date with the latest data. When we want to make improvements to how we generate documents, we can run a batch job that reindexes everything. Critically with Spark Streaming, the API is similar to the Spark API, so code is similar. It’s only at the top level where changes are necessary.

If you couldn’t tell already, we are huge fans of Spark. If you have any interest in big data we  would strongly encourage you to check it out. We got really lucky with the timing of starting a lot of these projects (right around the time that Spark 1.0 was released). Being able to build things from the ground up in Spark without having to port an existing Hadoop pipeline has been pretty cool and really exciting. If you’re the kind of person that also gets excited about the idea of building production applications with Spark, you should [check us out](http://www.workday.com/company/careers.php)!

### About Andrew

Andrew Bullen is a Senior Software engineer on the Syman team at Workday. The Syman team uses Scala and Spark to help Workday’s customers make sense of massive amounts of data.
