## Fabric8 Java Base Image {{= fp.config.version.description }} ({{= fp.param.type.toUpperCase() }})

This image is based on {{= fp.config.base.description }} and provides {{= fp.config.version.description }} ({{= fp.param.type.toUpperCase() }})

It includes:

* A startup script [`{{= fp.config.base.baseDir }}/run-java.sh`](#startup-script-run-javash) for starting up Java applications.
* [Jolokia](http://www.jolokia.org) agent installed in `/opt/jolokia`
* Prometheus [jmx_exporter](https://github.com/prometheus/jmx_exporter) agent installed in `/opt/jmx_exporter`.

### Startup Script run-java.sh

The default command for this image is [{{= fp.config.base.baseDir }}/run-java.sh](https://github.com/fabric8io/run-java-sh). Its purpose it to fire up Java applications which are provided as fat-jars, including all dependencies or more classical from a main class, where the classpath is build up from all jars within a directory.

For these images the variable **JAVA_APP_DIR** has the default value `{{= fp.config.base.baseDir }}`

{{= fp.block('run-java-sh','readme',{ 'fp-no-files' : true }) }}

### Monitoring Agents

In order to enable Jolokia and Prometheus exporter for your application you should use this image as a base image (via `FROM`) and use the output of `jolokia-opts` and `prometheus-opts` in your startup scripts to include it in for the Java startup options. You can also just use the provided startup script `/opt/run-java.sh` which does this automatically.

For example, the following snippet can be added to a script starting up your Java application

    # ...
    export JAVA_OPTIONS="$JAVA_OPTIONS $(/opt/jolokia/jolokia-opts) $(/opt/prometheus/prometheus-opts)"
    # .... use JAVA_OPTIONS when starting your app, e.g. as Tomcat does

The following versions and defaults are used:

* [Jolokia](http://www.jolokia.org) : version **{{= fp.version.jolokia }}** and port **8778**
* [jmx_exporter](https://github.com/prometheus/jmx_exporter): version **{{= fp.version.jmxExporter }}** and port **9779**

You can influence the behaviour of these config scripts by setting various environment variables.

{{= fp.block('jolokia','readme',{ 'fp-no-files' : true }) }}

{{= fp.block('jmxexporter','readme',{ 'fp-no-files' : true }) }}

### Versions:

* Base-Image: **{{= fp.config.base.description + " " + fp.config.base.version }}**
* Java: **{{= fp.config.version.description + " " + fp.config.version.version }}** ({{= fp.config.type.description }})
* Jolokia: **{{= fp.version.jolokia }}**
* jmx_exporter: **{{= fp.version.jmx_exporter }}**
