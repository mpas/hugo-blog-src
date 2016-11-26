+++
date = "2016-11-25T20:31:27+01:00"
tags = ["grails", "docker"]
title = "Dockerize your Grails application"
+++

Ever wanted to create a Docker image that contains your Grails application? You are lucky! It is easy to do so..

Let us first create a new Grails application. In the example we will create a basic application using the rest-profile.

**Prerequisite** : Docker is installed on your machine.

```console
$ grails create-app docker-test --profile rest-api
```

After the Grails application has been created, we will need to add the following files to our project.

* Dockerfile (determines what our Docker image will contain)
* docker-entrypoint.sh (script that is responsible for starting our Grails application)

`File: /src/main/docker/Dockerfile`
```Dockerfile
FROM openjdk:latest

# set environment options
ENV JAVA_OPTS="-Xms64m -Xmx256m -XX:MaxMetaspaceSize=128m"

RUN mkdir -p /app
WORKDIR /app

COPY /app/application.jar application.jar
COPY /app/docker-entrypoint.sh docker-entrypoint.sh

# Set file permissions
RUN chmod +x /app/docker-entrypoint.sh

# Set start script as enrypoint
ENTRYPOINT ["./docker-entrypoint.sh"]
```

`File: /src/main/docker/app/docker-entrypoint.sh`
```bash
#!/bin/bash
set -e

exec java ${JAVA_OPTS} -jar application.jar $@
# exec java ${JAVA_OPTS} -Dserver.port=${SERVER_PORT} -jar application.jar $@
```

Next step is to add the tasks to our `build.gradle` so it can generate the Docker image.

So add the following snippet to your `build.gradle` file!

`File: /build.gradle`
```groovy
String getDockerImageName() {
  "docker-test"
}

task buildDockerImage(type:Exec) {
    group = 'docker'
    description = 'Build a docker image'
    commandLine 'docker', 'build', '-f', 'build/docker/Dockerfile', '-t', "${dockerImageName}", 'build/docker'

    doFirst {
        println ">> Creating image: ${dockerImageName}"
        /* copy the generate war file to /build/docker/app */
        copy {
            from war.archivePath
            into 'build/docker/app/'
        }
        /* copy artifacts from src/main/docker/app into the build/docker/app */
        copy {
            from 'src/main/docker/app/'
            into 'build/docker/app'
        }
        /* copy Dockerfile from src/main/docker into the build/docker */
        copy {
            from('src/main/docker/') {
                include 'Dockerfile'
            }
            into 'build/docker'
        }
        /* rename war file to application.jar */
        file("build/docker/app/${war.archiveName}").renameTo("build/docker/app/application.jar")
    }
}
```

Now that we have everyting in place we can build the image and start the container,

Create the docker image using assemble target and buildDockerImage
```console
$ ./gradlew assemble buildDockerImage
```

Run a container based on the previous created image
```console
$ docker run -it --rm -p 8080:8080 docker-test
```

This will run the container in interactive mode (`-it`) and the container will be removed when we stop the container (`--rm`). Port 8080 in the container will be available on port 8080 on your host system (`-p 8080:8080`).

This will run the specified container and the endpoint will be available using your browser. Just visit http://localhost:8080
