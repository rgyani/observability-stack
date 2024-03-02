# Observability

Observability is a way to keep track of system and application health and performance in the cloud-native age in order to keep those systems and applications up and running.

The three core pillars of observability are 
1. traces
2. metrics
3. logs


# Pillar 1: Tracing

A trace starts with a request, which can also trigger downstream requests to other services, generating a tree structure of multiple requests. 
Tracing provides insight into the journey of requests across system components like APIs, load balancers, services, and databases.


Distributed tracing is the method of tracking the performance of application requests to monitor microservices, 
enabling developers, site reliability engineers (SREs), IT ops, and DevOps users to identify performance issues.

It is based on concepts Google published in its [Dapper paper](https://research.google/pubs/dapper-a-large-scale-distributed-systems-tracing-infrastructure/) back in 2010. 
The idea is to implicitly observe how a set of services interact with each other by having each service emit information about the actions it performed and enrich those signals with a common identifier.


### Implementing Tracing

There are many standards and implementations of tracing. There are currently three widely-used standards: 
1. Opentracing: The most widely used standard is Opentracing, started by CNCF. It supports plenty of programming languages. Meanwhile, it has already integrated with many widely used frameworks like spring, nginx and resttemplate.
2. Opencensus: Started by Google, it supports both tracing and metrics data. But compared with opentracing, it supports fewer frameworks and programming languages. 
3. Opentelemetry: It not only combines pros of both opentracing and opencensus, but also avoids their cons. 



# Pillar 2: Metrics

Metrics represent aggregate data points reflecting a system's operational state, including query rates, API responsiveness, and service latencies
Various TimeSeries databases like Influx can be used to store the data and visualized using Grafana

Prometheus is the most commonly used metrics-based open source monitoring solution.

[Prometheus documentation](https://prometheus.io/docs/introduction/overview/) provides this architecture diagram

![img](imgs/prometheus.png)

For most use cases, you should understand three major components of Prometheus:

1. The **Prometheus server** scrapes and stores metrics. Note that it uses a persistence layer, which is part of the server and not expressly mentioned in the documentation. Each node of the server is autonomous and does not rely on distributed storage. **So you will in most cases need a time-series database to store Prometheus data, rather than relying on the server itself**.
2. **The web UI** allows you to access, visualize, and chart the stored data. Prometheus provides its own UI, but **you can also configure other visualization tools, like Grafana**, to access the Prometheus server using PromQL (the Prometheus Query Language).
3. **Alertmanager** sends alerts from client applications, especially the Prometheus server. It has advanced features for deduplicating, grouping, and routing alerts and **can route through other services like PagerDuty and OpsGenie.**

The key to understanding Prometheus is that **it fundamentally relies on scraping, or pulling, metrics from defined endpoints. This means that your application needs to expose an endpoint where metrics are available and instruct the Prometheus server how to scrape it**.

There are exporters for many applications that do not have an easy way to add web endpoints, such as Kafka and Cassandra **(using the JMX exporter)**


# Pillar 3: Logs
Logging involves recording discrete events within a system, such as incoming requests, database accesses, time taken to process the request and obviously errors.
It typically generates high volumes of data. 

The ELK stack (Elasticsearch, Logstash, Kibana) is commonly used to build log analysis platforms. 

### What is the ELK Stack?
The ELK Stack began as a collection of three open-source products — Elasticsearch, Logstash, and Kibana — all developed, managed and maintained by Elastic. The introduction and subsequent addition of Beats turned the stack into a four legged project.

* **Elasticsearch** is a full-text search and analysis engine, based on the Apache Lucene open source search engine. 
* **Logstash** is a log aggregator that collects data from various input sources, executes different transformations and enhancements and then ships the data to various supported output destinations. It’s important to know that many modern implementations of ELK do not include Logstash. To replace its log processing capabilities, most turn to lightweight alternatives like Fluentd, which can also collect logs from data sources and forward them to Elasticsearch. 
* **Kibana** is a visualization layer that works on top of Elasticsearch, providing users with the ability to analyze and visualize the data. And last but not least — Beats are lightweight agents that are installed on edge hosts to collect different types of data for forwarding into the stack.
* **Beats** refers to a family of lightweight data shippers or agents developed by Elastic. Beats are designed to efficiently collect, ship, and parse log data or metrics from various sources to Elasticsearch or Logstash for indexing and analysis.

![](imgs/elk.png)


In early 2021, Elastic announced a bombshell in the open source world: [the ELK Stack would no longer be open source, as of version 7.11](https://logz.io/blog/open-source-elasticsearch-doubling-down/).   
As, I Understand, they did this because they were concerned about other big companies, like Amazon Web Services (AWS), using their open source software without contributing much back to the community. They felt that this wasn't fair, as they were investing a lot of resources into developing and maintaining the software.
In response, AWS created their own open-source version of Elasticsearch called OpenSearch (forked from ElasticSearch).

### OpenSearch Components

Source [OpenSearch Documentation](https://opensearch.org/docs/latest/data-prepper/common-use-cases/log-analytics/)

![](imgs/opensearch.jpg)

* **FluentBit** can be used as a component for log collection and forwarding. Fluent Bit can be containerized through Kubernetes, Docker, or Amazon Elastic Container Service (Amazon ECS). You can also run Fluent Bit as an agent on Amazon Elastic Compute Cloud (Amazon EC2). Configure the Fluent Bit http output plugin to export log data to Data Prepper.
* **Data Prepper** supports receiving logs from Fluent Bit through the HTTP Source and processing those logs with a Grok Processor before ingesting them into OpenSearch through the OpenSearch sink.

* ![](imgs/data-prepper.jpg)