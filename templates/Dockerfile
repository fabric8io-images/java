FROM {{= fp.config.base.from + (fp.config.base.tagFromMajorVersion ? ":" + fp.config.version.major : "") }}

USER root

RUN mkdir -p {{= fp.config.base.baseDir }}

# JAVA_APP_DIR is used by run-java.sh for finding the binaries
ENV JAVA_APP_DIR={{= fp.config.base.baseDir }} \
    JAVA_MAJOR_VERSION={{= fp.config.version.major }}

{{= fp.block("java-pkg-" + fp.param.base) || ''}}
{{= fp.block(fp.config.base.agent + "-install", { dest: "/opt/run-java-options" }) }}

{{= fp.block("run-java-sh","copy",{dest: fp.config.base.baseDir}) }}

{{? fp.config.base.user }}
# Run under user "{{= fp.config.base.user}}" and prepare for be running
# under OpenShift, too
RUN {{? fp.config.base.createUser }}groupadd -r {{= fp.config.base.user }} -g 1000 \
  && useradd -u 1000 -r -g {{= fp.config.base.user }} -m -d /opt/{{= fp.config.base.user}} -s /sbin/nologin {{= fp.config.base.user }} \
  && chmod 755 /opt/{{= fp.config.base.user }} \
  && {{?}}chown -R {{= fp.config.base.user}} {{= fp.config.base.baseDir }} \
  && usermod -g root -G `id -g {{= fp.config.base.user }}` {{= fp.config.base.user }} \
  && chmod -R "g+rwX" {{= fp.config.base.baseDir }} \
  && chown -R {{= fp.config.base.user}}:root {{= fp.config.base.baseDir }}

USER {{= fp.config.base.user }}
{{?}}
{{? fp.config.base.ubiUser }}
# Run under user "{{= fp.config.base.ubiUser }}" and prepare for be running
# under OpenShift, too
RUN chown {{= fp.config.base.ubiUser }} {{= fp.config.base.baseDir }} \
    && chmod "g+rwX" {{= fp.config.base.baseDir }} \
    && chown {{= fp.config.base.ubiUser }}:root {{= fp.config.base.baseDir }} \
    && chmod g+rw /etc/passwd \
    && mkdir -p {{= fp.config.base.baseDir }}/data \
    && chown {{= fp.config.base.ubiUser }} {{= fp.config.base.baseDir }}/run-java.sh

# Gives uid
USER {{= fp.config.base.ubiUser }}
{{?}}
CMD [ "{{= fp.config.base.baseDir }}/run-java.sh" ]
