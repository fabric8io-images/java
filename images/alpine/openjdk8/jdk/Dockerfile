FROM alpine:3.4

USER root

RUN mkdir -p /deployments

# JAVA_APP_DIR is used by run-java.sh for finding the binaries
ENV JAVA_APP_DIR=/deployments


# /dev/urandom is used as random source, which is prefectly safe
# according to http://www.2uo.de/myths-about-urandom/
RUN echo "http://dl-4.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
 && apk add --update \
    curl \
    openjdk8 \
 && rm /var/cache/apk/* \
 && echo "securerandom.source=file:/dev/urandom" >> /usr/lib/jvm/default-jvm/jre/lib/security/java.security

# Agent bond including Jolokia and jmx_exporter
ADD agent-bond-opts /opt/run-java-options
RUN mkdir -p /opt/agent-bond \
 && curl http://central.maven.org/maven2/io/fabric8/agent-bond-agent/0.1.4/agent-bond-agent-0.1.4.jar \
          -o /opt/agent-bond/agent-bond.jar \
 && chmod 444 /opt/agent-bond/agent-bond.jar \
 && chmod 755 /opt/run-java-options
ADD jmx_exporter_config.yml /opt/agent-bond/
EXPOSE 8778 9779

# Add run script as /deployments/run-java.sh and make it executable
COPY run-java.sh debug-options container-limits java-default-options /deployments/
RUN chmod 755 /deployments/run-java.sh /deployments/java-default-options /deployments/container-limits /deployments/debug-options




CMD [ "/deployments/run-java.sh" ]
