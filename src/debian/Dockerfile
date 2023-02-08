# This is free and unencumbered software released into the public domain.
# Anyone is free to copy, modify, publish, use, compile, sell, or distribute this
# software, either in source code form or as a compiled binary, for any purpose,
# commercial or non-commercial, and by any means.
# In jurisdictions that recognize copyright laws, the author or authors of this
# software dedicate any and all copyright interest in the software to the public
# domain. We make this dedication for the benefit of the public at large and to
# the detriment of our heirs and successors. We intend this dedication to be an
# overt act of relinquishment in perpetuity of all present and future rights to
# this software under copyright law.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
# INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
# PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS BE LIABLE
# FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
# TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR
# THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
# For more information, please refer to <https://unlicense.org/>

# Author: Leonardo Spaccini <leonardo.spaccini.gtr@gmail.com>


# Build stage which is used to download Java (requires wget) and to verify its checksum.
# Note: this stage uses the same base image of the final stage, so that it is not needed
# to download different base images when building this Dockerfile.
# Additionally, wget is installed before declaring the build arguments, so that the
# cached layer may be resued with different build arguments.
ARG BASE_IMAGE_VERSION
FROM debian:${BASE_IMAGE_VERSION} AS build-stage

RUN apt-get update -y && apt-get install wget -y

ARG JAVA_DOWNLOAD_URL
ARG JAVA_CHECKSUM_SHA256

RUN wget -4 $JAVA_DOWNLOAD_URL --no-verbose --output-document=/tmp/java-binaries.tar.gz \
    && echo "$JAVA_CHECKSUM_SHA256 /tmp/java-binaries.tar.gz" | sha256sum --check \
    && mkdir -p /tmp/java-binaries \
    && tar -xzf /tmp/java-binaries.tar.gz -C /tmp/java-binaries --strip-components=1 \
    && rm /tmp/java-binaries.tar.gz;


# Final stage
# Contains labels following the Open Container Initiative (https://opencontainers.org/).
FROM debian:${BASE_IMAGE_VERSION}

ARG BASE_IMAGE_VERSION
LABEL \
    org.opencontainers.image.title="Java on Debian" \
    org.opencontainers.image.description="A collection of Docker images based on \"debian:$BASE_IMAGE_VERSION\" that provides specific distributions and versions of the Java platform binaries." \
    org.opencontainers.image.url="https://github.com/iamryusei/java-on-docker" \
    org.opencontainers.image.version="1.0.0" \
    org.opencontainers.image.revision="" \
    org.opencontainers.image.created=""\
    org.opencontainers.image.authors="Leonardo Spaccini <leonardo.spaccini.gtr@gmail.com>" \
    org.opencontainers.image.license="Unlicense"

ARG JAVA_PLATFORM_NAME
ARG JAVA_HOME="/opt/java/$JAVA_PLATFORM_NAME"

ENV JAVA_HOME=$JAVA_HOME \
    PATH=$PATH:$JAVA_HOME/bin

COPY --from=build-stage /tmp/java-binaries $JAVA_HOME

CMD [ "java", "-version" ]