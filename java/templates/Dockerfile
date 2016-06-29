FROM {{= fp.config.base.from + (fp.config.base.tagFromMajorVersion ? ":" + fp.config.version.major : "") }}

USER root

RUN mkdir /app
ENV JAVA_APP_DIR /app

{{= fp.block("java-pkg-" + fp.param.base) || ''}}
{{= fp.block(fp.config.base.agent, "install", { dest: "/opt/run-java-options" }) }}
{{= fp.block("run-java-sh","copy",{dest: "/app"}) }}

{{? fp.config.base.user }}
# Run under user "{{= fp.config.base.user}}"
RUN chown -R {{= fp.config.base.user}} /app
USER {{= fp.config.base.user }}
{{?}}

CMD [ "/app/run-java.sh" ]
