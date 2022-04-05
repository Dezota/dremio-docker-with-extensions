#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
FROM registry.access.redhat.com/ubi8/ubi

# who's your boss?
MAINTAINER "Brian Holman" <bholman@dezota.com>

# Base Dremio OSS Version 20.1.0
ENV DREMIO_COMMIT 31bc9b7ce0aa88ac2ab8ebcbc9fb69fb0d2744a8
ENV CH_CONNECTOR_COMMIT 6297baf12694d1c8946044ef97e6d78add714e54
ENV GIS_EXT_COMMIT ef17bccc94f33dfb88b072325b351181f41e9410
ENV JAVA_HOME /usr/lib/jvm/zulu8

USER root

RUN \
    set -xeu && \
    yum -y module enable nodejs:14 && \
    yum -y -q install https://cdn.azul.com/zulu/bin/zulu-repo-1.0.0-1.noarch.rpm && \
    curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | tee /etc/yum.repos.d/yarn.repo && \
    rpm --import https://dl.yarnpkg.com/rpm/pubkey.gpg && \
    yum -y -q install git maven zulu8-jdk zulu11-jdk less python2 rpm-build nodejs yarn make gcc gcc-c++ && \
    yum -q clean all && \
    rm -rf /var/cache/yum
    
RUN mkdir -p /dremio-build

ENV DEZOTA_BUILD 11
    
COPY ./varchar_size.patch /dremio-build

# Build Dremio with patch to increase default VARCHAR/VARBINARY size to support reasonable GIS fields and add our GIS Funtions to the DAC search
# Build and install the RPM file for Dremio
# Build the Dremio Clickhouse Connector
# Build the Dremio GIS Extensions
RUN \
    git config --global user.email bholman@dezota.com && \
    git config --global user.name "Brian Holman" && \
    echo "%_unpackaged_files_terminate_build 0" > ~/.rpmmacros && \
    echo "%_binaries_in_noarch_packages_terminate_build 0" >> ~/.rpmmacros && \
    cd /dremio-build && \
    git clone --recursive https://github.com/dremio/dremio-oss.git && \
    cd dremio-oss && \
    git checkout $DREMIO_COMMIT && \
    git apply ../varchar_size.patch && \
    mvn clean install -DskipTests && \
    cd /dremio-build && \
    git clone https://github.com/Dezota/dremio-clickhouse-connector.git && \
    cd dremio-clickhouse-connector && \
    git checkout $CH_CONNECTOR_COMMIT && \
    mvn clean install -DskipTests && \
    cp ./target/dremio-clickhouse-plugin-20.1.0-202202061055110045-36733c65.jar /dremio-build/dremio-oss/distribution/server/target/dremio-community-20.1.0-202202061055110045-36733c65/dremio-community-20.1.0-202202061055110045-36733c65/jars/ && \
    cd /dremio-build && \
    git clone https://github.com/Dezota/dremio-gis-extensions.git && \
    cd dremio-gis-extensions && \
    git checkout $GIS_EXT_COMMIT && \
    mvn clean install -DskipTests && \
    cp ./target/dremio-gis-extensions-20.1.0-202202061055110045-36733c65.jar /dremio-build/dremio-oss/distribution/server/target/dremio-community-20.1.0-202202061055110045-36733c65/dremio-community-20.1.0-202202061055110045-36733c65/jars/ && \
    cp ./docker/build/h3-3.7.2.jar /dremio-build/dremio-oss/distribution/server/target/dremio-community-20.1.0-202202061055110045-36733c65/dremio-community-20.1.0-202202061055110045-36733c65/jars/3rdparty/ && \
    cd /dremio-build/dremio-oss/distribution && \
    mvn package -P rpm && \
    yum -y -q install /dremio-build/dremio-oss/distribution/server/target/rpm/dremio-community/RPMS/noarch/dremio-community-20.1.0-202202061055110045_36733c65_1.noarch.rpm && \
    mkdir -p /media && \
    cp /dremio-build/dremio-oss/distribution/server/target/rpm/dremio-community/RPMS/noarch/dremio-community-20.1.0-202202061055110045_36733c65_1.noarch.rpm /media/ && \
    rm -rf /root/.m2 && \
    rm -rf /dremio-build

RUN yum -y -q remove zulu8-jdk java-1.8.0-openjdk-headless.x86_64
	
ENV JAVA_HOME /usr/lib/jvm/zulu11
COPY ./startup.sh /opt/dremio/bin/startup.sh
# Uncomment this copy to enable full debug
COPY ./logback.xml /opt/dremio/conf/logback.xml
COPY ./dremio-bin /opt/dremio/bin/dremio
RUN chmod 755 /opt/dremio/bin/dremio && \
    chmod 755 /opt/dremio/bin/startup.sh && \
    chown -R dremio:dremio /opt/dremio && \
    chown -R dremio:dremio /usr/lib/jvm/zulu8-ca && \
    chown -R dremio:dremio /usr/lib/jvm/zulu11-ca 
EXPOSE 2181 9047 31010 45678
USER dremio:dremio
ENV LANG en_US.UTF-8
ENTRYPOINT ["/opt/dremio/bin/startup.sh"]
