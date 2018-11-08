+++
date = "2016-11-24T19:34:45+01:00"
tags = ["docker", "gradle", "groovy"]
title = "Creating/Pushing Docker images using Gradle without plugins"
+++

In our current project we where heavily focussed on the usage of Gradle plugins to create Docker images. We used plugins to create the images and push them to our AWS ECR repositories. When using these plugins we hit various bugs related to the fact that not all developers where using Linux operating systems to test our their containers. At the end we took a look on how we could create those images without using additional plugins.

**Prerequisite** : Docker is installed on your machine.

## Creating an image
The following snippet will create a Docker image using the task `gradle buildDockerImage`

    - application layout
    | build.gradle
    | > src
    | >- main
    | >-- docker (contains a Dockerfile)
    | >--- app (contains data that can be used in your Dockerfile)

```groovy
/*
    You can put some logic in the getDockerImageName to determine how your
    Docker image should be created.
*/
String getDockerImageName() {
  "my-first-docker-image"
}

/*
    Execute a docker build using commandline pointing to our Dockerfile that
    has been copied to /build/docker/.
*/
task buildDockerImage(type:Exec) {
    group = 'docker'
    description = 'Build a docker image'
    commandLine 'docker', 'build', '-f', 'build/docker/Dockerfile', '-t', "${dockerImageName}", 'build/docker'

    doFirst {
        println ">> Creating image: ${dockerImageName}"
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
    }
}
```

## pushing an image

Pushing an image without using plugins is just as easy.

```groovy
pushDockerImage(type: Exec) {
    group = 'docker'
    description = 'Push a docker image'
    commandLine 'docker', 'push', "${dockerImageName}"

    doFirst {
        println ">> Pushing image: ${dockerImageName}"
    }
}
```

Using this approach without using unneeded Gradle plugins resulted in a an easy way to create containers on different platforms.
