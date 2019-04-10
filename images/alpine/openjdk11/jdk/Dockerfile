FROM alpine:3.10

USER root

RUN mkdir -p /deployments

# JAVA_APP_DIR is used by run-java.sh for finding the binaries
ENV JAVA_APP_DIR=/deployments \
    JAVA_MAJOR_VERSION=11


# /dev/urandom is used as random source, which is perfectly safe
# according to http://www.2uo.de/myths-about-urandom/
RUN apk add --update \
    curl \
    openjdk11=11.0.4_p4-r1 \
 && rm /var/cache/apk/* \
 && echo "securerandom.source=file:/dev/urandom" >> /usr/lib/jvm/default-jvm/jre/lib/security/java.security

# Prometheus JMX exporter agent
RUN mkdir -p /opt/prometheus/etc \
 && curl http://central.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/undefined/jmx_prometheus_javaagent-undefined.jar \
          -o /opt/prometheus/jmx_prometheus_javaagent.jar
ADD prometheus-opts /opt/prometheus/prometheus-opts
ADD prometheus-config.yml /opt/prometheus/prometheus-config.yml
RUN chmod 444 /opt/prometheus/jmx_prometheus_javaagent.jar \
 && chmod 444 /opt/prometheus/prometheus-config.yml \
 && chmod 755 /opt/prometheus/prometheus-opts \
 && chmod 775 /opt/prometheus/etc \
 && chgrp root /opt/prometheus/etc

EXPOSE 9779


# Jolokia agent
RUN mkdir -p /opt/jolokia/etc \
 && curl http://central.maven.org/maven2/org/jolokia/jolokia-jvm/1.6.2/jolokia-jvm-1.6.2-agent.jar \
         -o /opt/jolokia/jolokia.jar
ADD jolokia-opts /opt/jolokia/jolokia-opts
RUN chmod 444 /opt/jolokia/jolokia.jar \
 && chmod 755 /opt/jolokia/jolokia-opts \
 && chmod 775 /opt/jolokia/etc \
 && chgrp root /opt/jolokia/etc

EXPOSE 8778


# Add run script as /deployments/run-java.sh and make it executable
COPY run-java.sh /deployments/
RUN chmod 755 /deployments/run-java.sh




CMD [ "/deployments/run-java.sh" ]
