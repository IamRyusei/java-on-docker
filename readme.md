# Java on Docker
A collection of Docker images that provides multiple distributions and versions of the Java platform binaries (JRE/JDK) for different base images.

[![Build, Test and Deploy Docker Images](https://github.com/iamryusei/java-on-docker/actions/workflows/workflow.yml/badge.svg?branch=master)](https://github.com/iamryusei/java-on-docker/actions/workflows/workflow.yml)

The purpose of this repository is to provide a collection of Docker images that can be used as a base to build Java applications
or workflows. It is possible to pull the images directly from a remote repository to create your container, or to use them as a
base image that can be customized in a custom Dockerfile.

_...but why would a repository bother to provide so many combinations of Java flavours as Docker images?_

The first reason is that many publicly available Docker images that provides Java don't actually provide
a lot of choice in terms of which Java vendor is provided (with most of them being vendor-specific images),
or which base OS is used, and offers a narrow range of Java versions to select from. Sometimes is not even
possible to pick just a JRE over a JDK, or vice versa.

This means that, for practical use cases such as multi-stage builds (which requires one JDK image for the build
and one JRE image for the execution) or application benchmarking (which requires to use one image for every JVM
implementation and version that is needed to be benchmarked against) required to depend from multiple Docker
image providers, that may not even provide the same version of Java or the same base OS to begin with, which may
make a multi-stage build configuration toughest, or a benchmarking result unreliable because of how different the
different environments acutally are.
On top of that, it's just annoying that there is no single naming convention when it comes to deciding tags for Docker images for Java.

The goal of this repository is to solve both of those problems by providing a common ground that anyone can use
as a starting point for their Docker images for Java.

You are of course encouraged to contribute to this repository, or fork it for your own needs.

## Getting Started with Image Tags
Because it would be cumbersome to list each and every available tag that is made available from this repository,
this paragraph will provide a few rules to understand the naming convention that was adopted, and to quickly
identify what exactly each image provides.

The tags of the Docker images that are built from this repository are always structured as it follows
(in lowercase):

`java-on-${base-image}:${java-distribution}-${java-version}-${java-type}`

Therefore, the first part of the tag is always used to describe which base image was used for the build
(including the base image version and flavour), while the second part describes which Java binaries are installed.

This naming convention is convenient because it is usually always possible to choose the base image and the
Java binaries (almost) **independently**, which means that any value of `${base-image}` is available with (almost)
any value of `${java-distribution}-${java-version}-${java-type}`.

However, regarding the second part of the tag, note that not every combination of `${java-distribution}`,
`${java-version}` and `${java-type}` is always available, so that selecting a proper combination may require
a little more attention.

It the next paragraphs it will be explained in details which are the possible values for each of the forementioned fields.

### ${base-image}
The available options to use for this field, at least currently, are:
- `alpine-3.17`
- `centos-7`
- `debian-10`
- `debian-10-slim`
- `debian-11`
- `debian-11-slim`
- `ubuntu-23.04`

### ${java-distribution}
The available options to use for this field, at least currently, are:
- `corretto`: the "Amazon Corretto" JVM is the Amazon's build of the OpenJDK sources, distributed under the [GPLv2 with CPE](https://openjdk.org/legal/gplv2+ce.html) license.
- `temurin`: the "Eclipse Temurin" JVM is the Eclipse Adoptium Working Group's build of the OpenJDK sources, distributed under the [GPLv2 with CPE](https://openjdk.org/legal/gplv2+ce.html) license.
- `zulu`: the "Zulu Community" JVM is the Azul Systems's build of the OpenJDK sources, distributed under their [terms of use](https://www.azul.com/products/core/openjdk-terms-of-use/).

### ${java-version}
### ${java-type}
The available options to use for this field, do actually depend on the option 
that was specified for the \
`${java-distribution}`.
The following table contains the possible options for any combination of the three parameters:

| ${java-distribution} | ${java-version} | ${java-type}                         |
| :------------------- | :-------------- | :----------------------------------- |
| `corretto`           | `8`             | `jdk`                                |
| `corretto`           | `11`            | `jdk`                                |
| `corretto`           | `17`            | `jdk`                                |
| `temurin`            | `8`             | `jre`, `jdk`                         |
| `temurin`            | `11`            | `jre`, `jdk`                         |
| `temurin`            | `17`            | `jre`, `jdk`                         |
| `zulu`               | `6`\*           | `jdk`                                |
| `zulu`               | `7`\*           | `jdk`                                |
| `zulu`               | `8`             | `jre`, `jre-fx`\*, `jdk`, `jdk-fx`\* |
| `zulu`               | `11`            | `jre`, `jre-fx`\*, `jdk`, `jdk-fx`\* |
| `zulu`               | `17`            | `jre`, `jre-fx`\*, `jdk`, `jdk-fx`\* |

[*] = not available for `alpine`

## Usage Example (Console)
Once that a specific Docker image tag has been identified you can verify that
the version of Java provided by that image is the one actually expected by pulling
the image directly from the GitHub Container Registry (ghcr.io) and opening an
interactive console to print the version of Java.

For example, issuing the following commands:
```
> docker pull ghcr.io/iamryusei/java-on-alpine-3.17:zulu-8-jre
> docker run --rm ghcr.io/iamryusei/java-on-alpine-3.17:zulu-8-jre
```

Will produce the following output:
```
openjdk version "1.8.0_362"
OpenJDK Runtime Environment (Zulu 8.68.0.21-CA-linux-musl-x64) (build 1.8.0_362-b09)
OpenJDK 64-Bit Server VM (Zulu 8.68.0.21-CA-linux-musl-x64) (build 25.362-b09, mixed mode)
```

## Usage Example (Dockerfile)
In most use cases, you want to extend the image with a custom one that
contains your Java-related tools and applications.

The following example demonstrates a `Dockerfile` that implements
an "Hello World" application in Java that is built (therefore a JDK-type
image is required) and executed inside the container.

First create the `Dockerfile` with the following content:
```Dockerfile
FROM ghcr.io/iamryusei/java-on-alpine-3.17:zulu-8-jdk

WORKDIR /opt/helloworld/
RUN echo "public class HelloWorld {             \
    public static void main(String[] args) {    \
        System.out.println(\"Hello, World!\");  \ 
    }}" >> HelloWorld.java
RUN javac HelloWorld.java
ENTRYPOINT java HelloWorld
```

Then, issuing the following commands:
```
> docker build -t helloworldjava .
> docker run --rm helloworldjava
```

Will produce the following output:
```
Hello, World!
```

**NOTICE**: for simplicity, this example runs the container as `root`.\
For security reasons, always remember to build yout images so that
their containers will run as an unpriviledged user instead.