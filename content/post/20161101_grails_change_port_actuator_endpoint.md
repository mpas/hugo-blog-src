+++
date = "2016-11-01T19:44:16+01:00"
tags = ["grails", "springboot"]
title = "Grails change the port of actuator endpoints"
+++

When using actuator endpoints to expose metrics, it may be useful to run the
metrics on a different port.

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
