## Fabric8 Java Base Image {{= fp.config.version.description }} ({{= fp.param.type.toUpperCase() }})

{{? fp.config.base.deprecated }}> ** This image is deprecated and not maintained anymore. Please select one of the other flavors (alpine, centos).**

-----
{{?}}

This image is based on {{= fp.config.base.description }} and provides {{= fp.config.version.description }} ({{= fp.param.type.toUpperCase() }})

It includes:

{{? fp.config.base.agent == "agent-bond" }}
* An [Agent Bond](https://github.com/fabric8io/agent-bond) agent with [Jolokia](http://www.jolokia.org) and Prometheus' [jmx_exporter](https://github.com/prometheus/jmx_exporter). The agent is installed as `/opt/agent-bond/agent-bond.jar`. See below for configuration options.
{{?? fp.config.base.agent == "jolokia" }}
*  A [Jolokia](http://www.jolokia.org) agent. See below how to configure this.
{{?}}

* A startup script [`{{= fp.config.base.baseDir }}/run-java.sh`](#startup-script-run-javash) for starting up Java applications.

### Agent Bond

In order to enable Jolokia for your application you should use this image as a base image (via `FROM`) and use the output of `agent-bond-opts` in your startup scripts to include it in for the Java startup options.

For example, the following snippet can be added to a script starting up your Java application

    # ...
    export JAVA_OPTIONS="$JAVA_OPTIONS $(agent-bond-opts)"
    # .... use JAVA_OPTIONS when starting your app, e.g. as Tomcat does

The following versions and defaults are used:

* [Jolokia](http://www.jolokia.org) : version **{{= fp.version.jolokia }}** and port **8778**
* [jmx_exporter](https://github.com/prometheus/jmx_exporter): version **{{= fp.version.jmxExporter }}** and port **9779**

You can influence the behaviour of `agent-bond-opts` by setting various environment variables:

{{= fp.block(fp.config.base.agent,'readme',{ 'fp-no-files' : true }) }}

### Startup Script run-java.sh

The default command for this image is [{{= fp.config.base.baseDir }}/run-java.sh](https://github.com/fabric8io/run-java-sh). Its purpose it to fire up Java applications which are provided as fat-jars, including all dependencies or more classical from a main class, where the classpath is build up from all jars within a directory.

For these images the variable **JAVA_APP_DIR** has the default value `{{= fp.config.base.baseDir }}`

{{= fp.block('run-java-sh','readme',{ 'fp-no-files' : true }) }}

### Versions:

* Base-Image: **{{= fp.config.base.description + " " + fp.config.base.version }}**
* Java: **{{= fp.config.version.description + " " + fp.config.version.version }}** ({{= fp.config.type.description }}){{? fp.config.base.agent == "agent-bond" }}
* Agent-Bond: **{{= fp.version.agentBond }}** (Jolokia {{= fp.version.jolokia }}, jmx_exporter {{= fp.version.jmxExporter }}){{?? fp.config.base.agent == "jolokia" }}
* Jolokia: **{{= fp.version.jolokia }}**
{{?}}
