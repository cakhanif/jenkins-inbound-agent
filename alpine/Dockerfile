ARG version=latest-alpine-jdk11
FROM jenkins/agent:$version

ARG version
ARG MAVEN_VERSION=3.8.6

ARG BASE_URL=https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries
ARG GCLOUD_URL=https://dl.google.com/dl/cloudsdk/release/google-cloud-sdk.tar.gz
ARG S2I_URL=https://github.com/openshift/source-to-image/releases/download/v1.3.1/source-to-image-v1.3.1-a5a77147-linux-amd64.tar.gz

LABEL Description="This is a base image, which allows connecting Jenkins agents via JNLP protocols" Vendor="Jenkins project" Version="$version"

ARG USER=jenkins
ENV MAVEN_HOME /usr/share/maven

USER root

RUN apk add --update --no-cache libgcc libstdc++ curl python3 nodejs npm docker openrc && ln -sf python3 /usr/bin/python

RUN mkdir -p /usr/share/maven /usr/share/maven \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn \
  && curl -fsSL -o s2i.tar.gz ${S2I_URL} \
  && tar -xzf s2i.tar.gz \
  && chmod +x ./s2i && mv ./s2i /usr/local/bin \
  && curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
  && chmod +x ./kubectl \
  && mv ./kubectl /usr/local/bin/kubectl \
  && mkdir -p /usr/local/gcloud \
  && curl -o google-cloud-sdk.tar.gz https://dl.google.com/dl/cloudsdk/release/google-cloud-sdk.tar.gz \
  && tar -xvf $(pwd)/google-cloud-sdk.tar.gz -C /usr/local/gcloud  \
  && /usr/local/gcloud/google-cloud-sdk/install.sh \
  && addgroup ${USER} docker

RUN rc-update add docker boot

ENV PATH $PATH:/usr/local/gcloud/google-cloud-sdk/bin

COPY alpine/jenkins-agent /usr/local/bin/jenkins-agent
RUN chmod +x /usr/local/bin/jenkins-agent \
    && ln -s /usr/local/bin/jenkins-agent /usr/local/bin/jenkins-slave \
    && chown -R ${USER}.${USER} /home/jenkins

ENTRYPOINT ["/usr/local/bin/jenkins-agent"]
