+++
date = "2016-11-01T19:44:16+01:00"
tags = ["grails", "springboot"]
title = "Change the port of actuator endpoint in a Grails application"
+++

When using actuator endpoints to expose metrics in a Grails (Spring Boot) application,
it may be useful to run the metrics on a different port.

This enables you to hide the metrics for the public and use the different port in
an AWS infrastucture so that the metrics are only available internal.

Let us first enable the actuator endpoints

```console
// File: grails-app/conf/application.yml

# Spring Actuator Endpoints are Disabled by Default
endpoints:
    enabled: true
    jmx:
        enabled: true
```

Change the port on which the metrics runs, add the lines below to the appl
```console
// File: grails-app/conf/application.yml

management:
    port: 9000
```

Now when you start your Grails application it run on port `8080` and the metrics
are available on port `9090`/metrics
