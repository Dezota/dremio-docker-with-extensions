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

# Base Dremio OSS Version 24.1.0
ENV DREMIO_COMMIT be47367c523e4eded35df4e8fff725f53160558e
ENV GIS_EXT_COMMIT 3be81aea205ef007be47a49ef65c6b01e26f0466
ENV JAVA_HOME /usr/lib/jvm/zulu8

USER root

RUN \
    set -xeu && \
    yum -y -q install https://cdn.azul.com/zulu/bin/zulu-repo-1.0.0-1.noarch.rpm && \
    curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | tee /etc/yum.repos.d/yarn.repo && \
    rpm --import https://dl.yarnpkg.com/rpm/pubkey.gpg && \
    yum -y -q install git zulu8-jdk zulu11-jdk less python2 rpm-build make gcc gcc-c++ && \
    yum -q clean all && \
    rm -rf /var/cache/yum

RUN mkdir -p /usr/share/maven /usr/share/maven/ref && \
    curl -fsSL -o /tmp/apache-maven.tar.gz https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz && \
    tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 && \
    rm -f /tmp/apache-maven.tar.gz && \
    ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
     
RUN mkdir -p /dremio-build

ENV DEZOTA_BUILD 1
    
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
    git clone https://github.com/Dezota/dremio-gis-extensions.git && \
    cd dremio-gis-extensions && \
    git checkout $GIS_EXT_COMMIT && \
    mvn clean install -DskipTests && \
    cp ./target/dremio-gis-extensions-24.1.0-202306130653310132-d30779f6.jar /dremio-build/dremio-oss/distribution/server/target/dremio-community-24.1.0-202306130653310132-d30779f6/dremio-community-24.1.0-202306130653310132-d30779f6/jars/ && \
    cp ./docker/build/h3-3.7.2.jar /dremio-build/dremio-oss/distribution/server/target/dremio-community-24.1.0-202306130653310132-d30779f6/dremio-community-24.1.0-202306130653310132-d30779f6/jars/3rdparty/ && \
    cd /dremio-build/dremio-oss/distribution && \
    mvn package -P rpm && \
    yum -y -q install /dremio-build/dremio-oss/distribution/server/target/rpm/dremio-community/RPMS/noarch/dremio-community-24.1.0-202306130653310132_d30779f6_1.noarch.rpm && \
    mkdir -p /media && \
    cp /dremio-build/dremio-oss/distribution/server/target/rpm/dremio-community/RPMS/noarch/dremio-community-24.1.0-202306130653310132_d30779f6_1.noarch.rpm /media/ && \
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
