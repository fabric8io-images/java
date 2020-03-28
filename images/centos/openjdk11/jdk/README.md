## Fabric8 Java Base Image OpenJDK 11 (JDK)



This image is based on CentOS and provides OpenJDK 11 (JDK)

It includes:


* An [Agent Bond](https://github.com/fabric8io/agent-bond) agent with [Jolokia](http://www.jolokia.org) and Prometheus' [jmx_exporter](https://github.com/prometheus/jmx_exporter). The agent is installed as `/opt/agent-bond/agent-bond.jar`. See below for configuration options.


* A startup script [`/deployments/run-java.sh`](#startup-script-run-javash) for starting up Java applications.

### Agent Bond

In order to enable Jolokia for your application you should use this image as a base image (via `FROM`) and use the output of `agent-bond-opts` in your startup scripts to include it in for the Java startup options.

For example, the following snippet can be added to a script starting up your Java application

    # ...
    export JAVA_OPTIONS="$JAVA_OPTIONS $(agent-bond-opts)"
    # .... use JAVA_OPTIONS when starting your app, e.g. as Tomcat does

The following versions and defaults are used:

* [Jolokia](http://www.jolokia.org) : version **1.6.2** and port **8778**
* [jmx_exporter](https://github.com/prometheus/jmx_exporter): version **0.3.1** and port **9779**

You can influence the behaviour of `agent-bond-opts` by setting various environment variables:

### Agent-Bond Options

Agent bond itself can be influenced with the following environment variables: 

* **AB_OFF** : If set disables activation of agent-bond (i.e. echos an empty value). By default, agent-bond is enabled.
* **AB_ENABLED** : Comma separated list of sub-agents enabled. Currently allowed values are `jolokia` and `jmx_exporter`. 
  By default both are enabled.


#### Jolokia configuration

* **AB_JOLOKIA_CONFIG** : If set uses this file (including path) as Jolokia JVM agent properties (as described 
  in Jolokia's [reference manual](http://www.jolokia.org/reference/html/agents.html#agents-jvm)).
  By default this is `/opt/jolokia/jolokia.properties`.
* **AB_JOLOKIA_HOST** : Host address to bind to (Default: `0.0.0.0`)
* **AB_JOLOKIA_PORT** : Port to use (Default: `8778`)
* **AB_JOLOKIA_USER** : User for authentication. By default authentication is switched off.
* **AB_JOLOKIA_HTTPS** : Switch on secure communication with https. By default self signed server certificates are generated
  if no `serverCert` configuration is given in `AB_JOLOKIA_OPTS`
* **AB_JOLOKIA_PASSWORD** : Password for authentication. By default authentication is switched off.
* **AB_JOLOKIA_ID** : Agent ID to use (`$HOSTNAME` by default, which is the container id)
* **AB_JOLOKIA_OPTS**  : Additional options to be appended to the agent opts. They should be given in the format 
  "key=value,key=value,..."

Some options for integration in various environments:

* **AB_JOLOKIA_AUTH_OPENSHIFT** : Switch on client authentication for OpenShift TLS communication. The value of this 
  parameter can be a relative distinguished name which must be contained in a presented client certificate. Enabling this
  parameter will automatically switch Jolokia into https communication mode. The default CA cert is set to 
  `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` 
  
#### jmx_exporter configuration

* **AB_JMX_EXPORTER_OPTS** : Configuration to use for `jmx_exporter` (in the format `<port>:<path to config>`)
* **AB_JMX_EXPORTER_PORT** : Port to use for the JMX Exporter. Default: `9779`
* **AB_JMX_EXPORTER_CONFIG** : Path to configuration to use for `jmx_exporter`: Default: `/opt/agent-bond/jmx_exporter_config.json`



### Startup Script run-java.sh

The default command for this image is [/deployments/run-java.sh](https://github.com/fabric8io/run-java-sh). Its purpose it to fire up Java applications which are provided as fat-jars, including all dependencies or more classical from a main class, where the classpath is build up from all jars within a directory.

For these images the variable **JAVA_APP_DIR** has the default value `/deployments`


### run-java.sh

This general purpose startup script is optimized for running Java application from within containers. It is called like

```
./run-java.sh <sub-command> <options>
```
`run-java.sh` knows two sub-commands:

* `options` to print out JVM option which can be used for own invocation of Java apps (like Maven or Tomcat). It respects container constraints and includes all magic which is used by this script
* `run` executes a Java application as described below. This is also the default command so you can skip adding this command.

### Running a Java application

When no subcommand is given (or when you provide the default subcommand `run`), then by default this scripts starts up Java application.

The startup process is configured mostly via environment variables:

* **JAVA_APP_DIR** the directory where the application resides. All paths in your application are relative to this directory. By default it is the same directory where this startup script resides.
* **JAVA_LIB_DIR** directory holding the Java jar files as well an optional `classpath` file which holds the classpath. Either as a single line classpath (colon separated) or with jar files listed line-by-line. If not set **JAVA_LIB_DIR** is the same as **JAVA_APP_DIR**.
* **JAVA_OPTIONS** options to add when calling `java`
* **JAVA_MAJOR_VERSION** a number >= 7. If the version is set then only options suitable for this version are used. When set to 7 options known only to Java > 8 will be removed. For versions >= 10 no explicit memory limit is calculated since Java >= 10 has support for container limits. If omitted, this parameter's value will be guessed using the `JAVA_VERSION` variable, the `release` file or parsing the `java -version` command output.
* **JAVA_MAX_MEM_RATIO** is used when no `-Xmx` option is given in `JAVA_OPTIONS`. This is used to calculate a default maximal Heap Memory based on a containers restriction. If used in a Docker container without any memory constraints for the container then this option has no effect. If there is a memory constraint then `-Xmx` is set to a ratio of the container available memory as set here. The default is `25` when the maximum amount of memory available to the container is below 300M, `50` otherwise, which means in that case that 50% of the available memory is used as an upper boundary. You can skip this mechanism by setting this value to 0 in which case no `-Xmx` option is added.
* **JAVA_INIT_MEM_RATIO** is used when no `-Xms` option is given in `JAVA_OPTIONS`. This is used to calculate a default initial Heap Memory based on a containers restriction. If used in a Docker container without any memory constraints for the container then this option has no effect. If there is a memory constraint then `-Xms` is set to a ratio of the container available memory as set here. By default this value is not set.
* **JAVA_MAX_CORE** restrict manually the number of cores available which is used for calculating certain defaults like the number of garbage collector threads. If set to 0 no base JVM tuning based on the number of cores is performed.
* **JAVA_DIAGNOSTICS** set this to get some diagnostics information to standard out when things are happening
* **JAVA_MAIN_CLASS** A main class to use as argument for `java`. When this environment variable is given, all jar files in `$JAVA_APP_DIR` are added to the classpath as well as `$JAVA_LIB_DIR`.
* **JAVA_APP_JAR** A jar file with an appropriate manifest so that it can be started with `java -jar` if no `$JAVA_MAIN_CLASS` is set. In all cases this jar file is added to the classpath, too.
* **JAVA_APP_NAME** Name to use for the process
* **JAVA_CLASSPATH** the classpath to use. If not given, the startup script checks for a file `${JAVA_APP_DIR}/classpath` and use its content literally as classpath. If this file doesn't exists all jars in the app dir are added (`classes:${JAVA_APP_DIR}/*`).
* **JAVA_DEBUG** If set remote debugging will be switched on
* **JAVA_DEBUG_SUSPEND** If set enables suspend mode in remote debugging
* **JAVA_DEBUG_PORT** Port used for remote debugging. Default: 5005
* **HTTP_PROXY** The URL of the proxy server that translates into the `http.proxyHost` and `http.proxyPort` system properties.
* **HTTPS_PROXY** The URL of the proxy server that translates into the `https.proxyHost` and `https.proxyPort` system properties.
* **no_proxy**, **NO_PROXY** The list of hosts that should be reached directly, bypassing the proxy, that translates into the `http.nonProxyHosts` system property.

If neither `$JAVA_APP_JAR` nor `$JAVA_MAIN_CLASS` is given, `$JAVA_APP_DIR` is checked for a single JAR file which is taken as `$JAVA_APP_JAR`. If no or more then one jar file is found, an error is thrown.

The classpath is build up with the following parts:

* If `$JAVA_CLASSPATH` is set, this classpath is taken.
* The current directory (".") is added first.
* If the current directory is not the same as `$JAVA_APP_DIR`, `$JAVA_APP_DIR` is added.
* If `$JAVA_MAIN_CLASS` is set, then
  - A `$JAVA_APP_JAR` is added if set
  - If a file `$JAVA_APP_DIR/classpath` exists, its content is appended to the classpath. This file
    can be either a single line with the jar files colon separated or a multi-line file where each line
    holds the path of the jar file relative to `$JAVA_LIB_DIR` (which by default is the `$JAVA_APP_DIR`)
  - If this file is not set, a `${JAVA_APP_DIR}/*` is added which effectively adds all
    jars in this directory in alphabetical order.

These variables can be also set in a shell config file `run-env.sh`, which will be sourced by the startup script. This file can be located in the directory where the startup script is located and in `${JAVA_APP_DIR}`, whereas environment variables in the latter override the ones in `run-env.sh` from the script directory.

This startup script also checks for a command `run-java-options`. If existent it will be called and the output is added to the environment variable `$JAVA_OPTIONS`.

The startup script also exposes some environment variables describing container limits which can be used by applications:

* **CONTAINER_CORE_LIMIT** a calculated core limit as described in https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt
* **CONTAINER_MAX_MEMORY** memory limit given to the container

Any arguments given to the script are given through directly as argument to the Java application.

Example:

```
# Set the application directory directly
export JAVA_APP_DIR=/deployments
# Set -Xmx based on container constraints
export JAVA_MAX_MEM_RATIO=40
# Start the jar in JAVA_APP_DIR with the given arguments
./run-java.sh --user maxmorlock --password secret
```

### Options

This script can also be used to calculate reasonable, best-practice options for starting Java apps in general. For example, when running Maven in a container it makes sense to respect  container Memory constraints.

The subcommand `options` can be used to print options to standard output so that is can be easily used to feed it to another, Java based application.

When no extra arguments are given, all defaults will be used, which can be influenced with the environment variables described above.

You can select specific sets of options by providing additional arguments:

* `--debug` : Java debug options if `JAVA_DEBUG` is set
* `--memory` : Memory settings based on the environment variables given
* `--proxy` : Evaluate proxy environments variables
* `--cpu` : Tuning when the number of cores is limited
* `--gc` : GC tuning parameters
* `--jit` : JIT options
* `--diagnostics` : Print diagnostics options when `JAVA_DIAGNOSTICS` is set
* `--java-default` : Same as `--memory --jit --diagnostic --cpu --gc`

Example:

```
# Call Maven with the proper memory settings when running in an container
export MAVEN_OPTS="$(run-java.sh options --memory)"
mvn clean install
```


### Versions:

* Base-Image: **CentOS 7**
* Java: **OpenJDK 11 11** (Java Development Kit (JDK))
* Agent-Bond: **1.2.0** (Jolokia 1.6.2, jmx_exporter 0.3.1)
