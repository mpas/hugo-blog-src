+++
title = "Identifying Docker container outage using Prometheus"
tags = ["prometheus", "docker", "cadvisor"]
date = "2016-11-17"
+++

[Prometheus](https://prometheus.io/) is a an open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

Metric data is **pulled** (on a regular time-interval) from so called exporters which expose the metrics coming from applications/operating systems etc..

```console
+------------------+                               +----------+     Visualize data
|  +------------+  |                               | Grafana  +---> coming from
|  | Dockerized |  |                               +----+-----+     Prometheus
|  | Application|  |                                    |
|  +------------+  |                                    ^
|  +------------+  |  Pull data   +----------+          |
|  |  CAdvisor  +--------->-------+Prometheus+----------+
|  +------------+  |              +---------++
|                  |                        |
| Operating System |                        |
|       with       |                        |
| Docker installed |                        |
|                  |                        v
+------------------+           Prometheus collects data
                               coming from remote systems
```
In the diagram above cAdvisor is a so called exporter. There are other exporters like e.g. [Node Exporter](https://github.com/prometheus/node_exporter) that exposes machine metrics. **cAdvisor** is used to get Docker container metrics.

## cAdvisor
[cAdvisor](https://github.com/google/cadvisor) is a project coming from Google and analyzes resource usage and performance characteristics of running Docker  containers! When running a Dockerized application and starting a cAdvisor container you will have instant metrics available for all running containers.

A lot of metrics are exposed by cAdvisor of which one is the metric `container_last_seen`. You can use this metric in Prometheus to identify if a container has left the building :) The challenge with Prometheus is that it keeps the data for a specific amount of time the so called `Stale Timeout`. This means that Prometheus will keep reporting back that the data has been received until this timeout has occurred (default 5 minutes). This is of course too much if we need to identify if a container has gone.

So if you would normally query like this:

```sql
count(container_last_seen{job="<jobname>",name=~".*<containername>.*"})
```

This would get results until 5 minutes.. way to far...

A simple alternate query to identify if the container has gone is like below:
```sql
count(time() - container_last_seen{job="<jobname>",name=~".*<containername>.*"} < 30) OR vector(0)
```

The '30' is the time in seconds before we want to be notified if a container has gone. This time has to be larger then the pull interval for your job.

When using the mentioned query you can create a nice [Singlestat](http://docs.grafana.org/reference/singlestat/) panel in Grafan so you can display an alert when the container is gone.
