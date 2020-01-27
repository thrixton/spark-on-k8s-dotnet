# References
# https://github.com/debuerreotype/docker-debian-artifacts/blob/dist-amd64/buster/slim/Dockerfile
# https://github.com/docker-library/openjdk/blob/master/8/jdk/slim/Dockerfile
# https://github.com/dotnet/dotnet-docker/blob/master/2.1/runtime-deps/stretch-slim/amd64/Dockerfile
# https://github.com/dotnet/dotnet-docker/blob/master/2.2/runtime/stretch-slim/amd64/Dockerfile
# https://github.com/dotnet/dotnet-docker/blob/master/2.2/aspnet/stretch-slim/amd64/Dockerfile
# https://github.com/apache/spark/blob/master/resource-managers/kubernetes/docker/src/main/dockerfiles/spark/Dockerfile
# https://github.com/apache/spark/blob/master/resource-managers/kubernetes/docker/src/main/dockerfiles/spark/bindings/R/Dockerfile
# https://github.com/apache/spark/blob/master/resource-managers/kubernetes/docker/src/main/dockerfiles/spark/bindings/python/Dockerfile
# https://github.com/dotnet/spark/blob/master/docs/getting-started/ubuntu-instructions.md
# https://docs.microsoft.com/en-us/dotnet/core/linux-prerequisites?tabs=netcore2x

# All args are declared in the ../docker-compose.yaml file

# Spark args
ARG SPARK_VERSION
ARG SPARK_SUFFIX
ARG SPARK_INSTALL_DIR=/opt/spark-${SPARK_VERSION}

# Hadoop args
ARG HADOOP_VERSION
ARG HADOOP_AZURE_STORAGE_VERSION
ARG HADOOP_INSTALL_DIR=/opt/hadoop-${HADOOP_VERSION}

# Java args
ARG JAVA_VERSION

# dotnet args
ARG DOTNET_PACKAGE
ARG DOTNET_SDK_PACKAGE

# dotnet spark args
ARG SPARK_DOTNET_VERSION
ARG SPARK_DOTNET_TARGET_FRAMEWORK
ARG SPARK_DOTNET_INSTALL_DIR=/opt/spark-dotnet-${SPARK_DOTNET_VERSION}

# Prometheus args
ARG PROMETHEUS_EXPORTER_VERSION


# References
# https://dotnet.microsoft.com/download/linux-package-manager/debian9/sdk-2.2.401
FROM openjdk:${JAVA_VERSION}-jdk-slim-buster as build

# Spark args
ARG SPARK_VERSION
ARG SPARK_SUFFIX
ARG SPARK_FILE=spark-${SPARK_VERSION}-bin-${SPARK_SUFFIX}.tgz
ARG SPARK_URL=http://apache.mirror.amaze.com.au/spark/spark-${SPARK_VERSION}/${SPARK_FILE}


# Hadoop args
ARG HADOOP_VERSION
ARG HADOOP_FILE=hadoop-${HADOOP_VERSION}.tar.gz
ARG HADOOP_URL=http://apache.mirror.amaze.com.au/hadoop/common/hadoop-${HADOOP_VERSION}/${HADOOP_FILE}

# dotnet args
ARG DOTNET_SDK_PACKAGE

# dotnet spark args
ARG SPARK_DOTNET_VERSION
ARG SPARK_DOTNET_TARGET_FRAMEWORK
ARG SPARK_DOTNET_FILE=Microsoft.Spark.Worker.${SPARK_DOTNET_TARGET_FRAMEWORK}.linux-x64-${SPARK_DOTNET_VERSION}.tar.gz
ARG SPARK_DOTNET_SOURCE_FILE=v${SPARK_DOTNET_VERSION}.tar.gz
ARG SPARK_DOTNET_URL=https://github.com/dotnet/spark/releases/download/v${SPARK_DOTNET_VERSION}/${SPARK_DOTNET_FILE}
ARG SPARK_DOTNET_SOURCE_URL=https://github.com/dotnet/spark/archive/${SPARK_DOTNET_SOURCE_FILE}
ARG DELTA_CORE_VERSION
ARG SCALA_BINARY_VERSION
ARG MAVEN_BASE_URL
ARG DELTA_CORE_JAR=delta-core_${SCALA_BINARY_VERSION}-${DELTA_CORE_VERSION}.jar


# Linux vars
ENV DEBIAN_FRONTEND=noninteractive

# dotnet vars
ENV DOTNET_ROOT=/usr/share/dotnet \
    ASPNETCORE_URLS=http://+:80 \
    DOTNET_RUNNING_IN_CONTAINER=true \
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    NUGET_XMLDOC_MODE=skip

# Linux update
RUN set -eux && \
    apt-get update && \
    apt-get install -y \
        apt-utils && \
    apt-get upgrade -y
WORKDIR /build

# Spark deps
RUN apt-get install -y --no-install-recommends \
        wget curl

# dotnet deps
RUN apt-get install -y \
        apt-transport-https gnupg && \
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg && \
    mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/ && \
    wget -q https://packages.microsoft.com/config/debian/10/prod.list && \
    mv prod.list /etc/apt/sources.list.d/microsoft-prod.list && \
    chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg && \
    chown root:root /etc/apt/sources.list.d/microsoft-prod.list && \
    apt-get update && \
    apt-get install -y \
        ${DOTNET_SDK_PACKAGE} && \
    dotnet --info

ARG SPARK_INSTALL_DIR

# Spark install
RUN mkdir -p ${SPARK_INSTALL_DIR} && \
    wget ${SPARK_URL} && \
    tar xvzf ${SPARK_FILE} -C ${SPARK_INSTALL_DIR} --strip-components=1 && \
    rm -f ${SPARK_FILE}

# Delta.io support
RUN wget ${MAVEN_BASE_URL}/io/delta/delta-core_${SCALA_BINARY_VERSION}/${DELTA_CORE_VERSION}/${DELTA_CORE_JAR} && \
    mv ./${DELTA_CORE_JAR} ${SPARK_INSTALL_DIR}/jars/

ARG HADOOP_INSTALL_DIR

# Hadoop install
RUN mkdir -p ${HADOOP_INSTALL_DIR} && \
    wget ${HADOOP_URL} && \
    tar xvzf ${HADOOP_FILE} -C ${HADOOP_INSTALL_DIR} --strip-components=1 && \
    rm -f ${HADOOP_FILE}

ARG SPARK_DOTNET_INSTALL_DIR

# dotnet spark install
RUN mkdir -p ${SPARK_DOTNET_INSTALL_DIR} && \
    wget ${SPARK_DOTNET_URL} && \
    tar xvzf ${SPARK_DOTNET_FILE} -C ${SPARK_DOTNET_INSTALL_DIR} --strip-components=1 && \
    rm -f ${SPARK_DOTNET_FILE} && \
    cd ${SPARK_DOTNET_INSTALL_DIR} && \
    dotnet new classlib && \
    dotnet add package Microsoft.Spark --version ${SPARK_DOTNET_VERSION} && \
    dotnet add package Microsoft.Spark.Experimental --version ${SPARK_DOTNET_VERSION} && \
    dotnet publish -c Release -r debian.10-x64

# dotnet spark source install
RUN mkdir -p spark/dotnet && \
    wget ${SPARK_DOTNET_SOURCE_URL} && \
    tar xvzf ${SPARK_DOTNET_SOURCE_FILE} -C spark/dotnet --strip-components=1 && \
    rm -f ${SPARK_DOTNET_SOURCE_FILE}

# dotnet spark build
WORKDIR /build/spark/dotnet
RUN cd examples/Microsoft.Spark.CSharp.Examples && \
    dotnet publish -c Release -f ${SPARK_DOTNET_TARGET_FRAMEWORK} -r debian.10-x64 && \
    cd ../Microsoft.Spark.FSharp.Examples && \
    dotnet publish -c Release -f ${SPARK_DOTNET_TARGET_FRAMEWORK} -r debian.10-x64

# References
# https://dotnet.microsoft.com/download/linux-package-manager/debian9/runtime-2.2.6
# https://github.com/GoogleCloudPlatform/spark-on-k8s-operator/blob/master/spark-docker/Dockerfile
FROM openjdk:${JAVA_VERSION}-jdk-slim-buster as runtime

# dotnet args
ARG DOTNET_PACKAGE
ARG SPARK_DOTNET_INSTALL_DIR
ARG SPARK_DOTNET_VERSION

# dotnet spark args
ARG SPARK_DOTNET_TARGET_FRAMEWORK

# Prometheus args
ARG PROMETHEUS_EXPORTER_VERSION

# Spark args
ARG SPARK_INSTALL_DIR

# Spark vars
ENV SPARK_HOME=${SPARK_INSTALL_DIR}
ENV PATH=$SPARK_HOME/bin:$PATH

# Hadoop args
ARG HADOOP_INSTALL_DIR

# Hadoop vars
ENV HADOOP_HOME=${HADOOP_INSTALL_DIR}
ENV PATH=$HADOOP_HOME/bin:$PATH

# Python vars
ENV PYTHONPATH=${SPARK_HOME}/python/lib/pyspark.zip:${SPARK_HOME}/python/lib/py4j-*.zip

# R vars
ENV R_HOME=/usr/lib/R

# dotnet vars
ENV DOTNET_ROOT=/usr/share/dotnet \
    ASPNETCORE_URLS=http://+:80 \
    DOTNET_RUNNING_IN_CONTAINER=true \
    DOTNET_USE_POLLING_FILE_WATCHER=true

# dotnet spark vars
ENV DOTNET_WORKER_DIR=${SPARK_HOME}/dotnet-worker-${SPARK_DOTNET_VERSION}
ENV SPARKDOTNET_ROOT=${DOTNET_WORKER_DIR} \
    DOTNET_WORKER=${DOTNET_WORKER_DIR}/Microsoft.Spark.Worker

# Linux update
RUN set -eux && \
    apt-get update && \
    apt-get upgrade -y

# Linux update
RUN set -eux && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
    apt-utils && \
    apt-get autoremove -y
WORKDIR /build

# Spark deps
RUN apt-get install -y --no-install-recommends \
        bash tini libnss3 libpam-modules wget curl

# Python deps
RUN apt-get install -y --no-install-recommends \
        python python-setuptools python-pip \
        python3 python3-setuptools python3-pip && \
    python --version && \
    pip --version && \
    python2 --version && \
    pip2 --version && \
    python3 --version && \
    pip3 --version

# R deps
RUN apt-get install -y --no-install-recommends \
        r-base r-base-dev && \
    R --version

# dotnet deps
RUN apt-get install -y --no-install-recommends \
        apt-transport-https gnupg && \
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg && \
    mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/ && \
    wget -q https://packages.microsoft.com/config/debian/10/prod.list && \
    mv prod.list /etc/apt/sources.list.d/microsoft-prod.list && \
    chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg && \
    chown root:root /etc/apt/sources.list.d/microsoft-prod.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        ${DOTNET_PACKAGE} && \
    dotnet --info

# Python pam setup
RUN echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd

# Replace sh with bash and fix tini
RUN rm -f /bin/sh && \
    ln -svf /bin/bash /bin/sh && \
    ln -svf /usr/bin/tini /sbin/tini

# Hadoop install
RUN mkdir -p ${HADOOP_INSTALL_DIR}
COPY --from=build ${HADOOP_INSTALL_DIR} ${HADOOP_INSTALL_DIR}

# Spark install
RUN mkdir -p ${SPARK_INSTALL_DIR}/work-dir && \
    ln -svf /usr/bin/dotnet ${SPARK_INSTALL_DIR}/work-dir/dotnet && \
    touch $SPARK_HOME/RELEASE
COPY --from=build ${SPARK_INSTALL_DIR}/kubernetes/dockerfiles/spark/entrypoint.sh /opt/

COPY --from=build ${SPARK_INSTALL_DIR} ${SPARK_HOME}

# TEMP - Pending config fixes
#RUN spark-submit --version

# dotnet spark install
COPY --from=build ${SPARK_DOTNET_INSTALL_DIR}/bin/Release/netstandard2.0/debian.10-x64/publish/microsoft-spark-2.4.x-*.jar ${SPARK_HOME}/jars
COPY --from=build ${SPARK_DOTNET_INSTALL_DIR} ${DOTNET_WORKER_DIR}
COPY --from=build /build/spark/dotnet/artifacts/bin/Microsoft.Spark.CSharp.Examples/Release/${SPARK_DOTNET_TARGET_FRAMEWORK}/debian.10-x64/publish ${SPARK_HOME}/work-dir/Microsoft.Spark.CSharp.Examples
COPY --from=build /build/spark/dotnet/artifacts/bin/Microsoft.Spark.FSharp.Examples/Release/${SPARK_DOTNET_TARGET_FRAMEWORK}/debian.10-x64/publish ${SPARK_HOME}/work-dir/Microsoft.Spark.FSharp.Examples
RUN chmod +x ${DOTNET_WORKER} && \
    ln -svf ${DOTNET_WORKER} /usr/local/bin/Microsoft.Spark.Worker

# Setup for the Prometheus JMX exporter.
RUN mkdir -p /etc/metrics/conf
ADD https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${PROMETHEUS_EXPORTER_VERSION}/jmx_prometheus_javaagent-${PROMETHEUS_EXPORTER_VERSION}.jar /prometheus/
COPY prometheus-conf/metrics.properties /etc/metrics/conf
COPY prometheus-conf/prometheus.yaml /etc/metrics/conf

# Entrypoint
WORKDIR ${SPARK_HOME}/work-dir
ENTRYPOINT [ "/opt/entrypoint.sh" ]
