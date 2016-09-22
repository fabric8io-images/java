FROM {{= fp.config.base.from + (fp.config.base.tagFromMajorVersion ? ":" + fp.config.version.major : "") }}

USER root

RUN mkdir -p {{= fp.config.base.baseDir }}

# JAVA_APP_DIR is used by run-java.sh for finding the binaries
ENV JAVA_APP_DIR={{= fp.config.base.baseDir }}

{{= fp.block("java-pkg-" + fp.param.base) || ''}}
{{= fp.block(fp.config.base.agent, "install", { dest: "/opt/run-java-options" }) }}
{{= fp.block("run-java-sh","copy",{dest: fp.config.base.baseDir}) }}

{{? fp.config.base.user }}
# Run under user "{{= fp.config.base.user}}" and prepare for be running
# under OpenShift, too
RUN chown -R {{= fp.config.base.user}} {{= fp.config.base.baseDir }} \
 && usermod -g root -G `id -g {{= fp.config.base.user }}` {{= fp.config.base.user }} \
 && chmod -R "g+rwX" {{= fp.config.base.baseDir }} \
 && chown -R {{= fp.config.base.user}}:root {{= fp.config.base.baseDir }}

USER {{= fp.config.base.user }}
{{?}}

CMD [ "{{= fp.config.base.baseDir }}/run-java.sh" ]
