FROM alpine:3.4

USER root

RUN mkdir -p /app/data
RUN chmod 1777 /app/data
ENV JAVA_APP_DIR=/app \
    JAVA_DATA_DIR=/app/data


RUN echo "http://dl-4.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
 && apk add --update \
    curl \
    openjdk8 \
 && rm /var/cache/apk/*

# Agent bond including Jolokia and jmx_exporter
ADD agent-bond-opts /opt/run-java-options
RUN mkdir -p /opt/agent-bond \
 && curl http://central.maven.org/maven2/io/fabric8/agent-bond-agent/0.1.4/agent-bond-agent-0.1.4.jar \
          -o /opt/agent-bond/agent-bond.jar \
 && chmod 444 /opt/agent-bond/agent-bond.jar \
 && chmod 755 /opt/run-java-options
ADD jmx_exporter_config.yml /opt/agent-bond/
EXPOSE 8778 9779

# Add run script as /app/run-java.sh and make it executable
COPY run-java.sh debug-options java-container-options /app/
RUN chmod 755 /app/run-java.sh /app/java-container-options /app/debug-options




CMD [ "/app/run-java.sh" ]
