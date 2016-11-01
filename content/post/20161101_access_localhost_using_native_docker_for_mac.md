+++
date = "2016-11-01T19:05:44+01:00"
tags = ["docker", "osx", "mac"]
title = "Accessing localhost from a Docker Container"
+++

Ever had a need to access something from within a Docker that runs on the host system?

When using native Docker on OSX you have bad luck. When configuring a container and pointing
that to localhost will result in the fact the your software will be targeted at the localhost
of the docker container.

A solution for this isto define a new local loopback to your localhost

```shell
$ sudo ifconfig lo0 alias 172.16.123.1
```

This will define a loopback network interface that points to your localhost.
When you need to access the localhost you can use this ip.
