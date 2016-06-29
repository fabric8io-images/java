## Fabric8 Java Base Image OpenJDK 7 (JDK)

This image is based on CentOS and provides
OpenJDK 7 (JDK)

It includes:


* An [Agent Bond](https://github.com/fabric8io/agent-bond) agent with [Jolokia](http://www.jolokia.org) 
  and Prometheus' [jmx_exporter](https://github.com/prometheus/jmx_exporter). 
  The agent is installed as `/opt/agent-bond/agent-bond.jar`. See below for configuration options. 


* A startup script [`/app/run-java.sh`](#startup-script-run-javash) for starting up Java applications.

### Agent Bond

In order to enable Jolokia for your application you should use this 
image as a base image (via `FROM`) and use the output of `agent-bond-opts` in 
your startup scripts to include it in for the Java startup options. 

For example, the following snippet can be added to a script starting up your 
Java application

    # ...
    export JAVA_OPTIONS="$JAVA_OPTIONS $(agent-bond-opts)"
    # .... use JAVA_OPTIONS when starting your app, e.g. as Tomcat does

The following versions and defaults are used:

* [Jolokia](http://www.jolokia.org) : version **undefined** and port **8778**
* [jmx_exporter](https://github.com/prometheus/jmx_exporter): version **undefined** and port **9779**  

You can influence the behaviour of `agent-bond-opts` by setting various environment 
variables:

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

* **AB_JOLOKIA_AUTH_OPENSHIFT** : Switch on client authentication for OpenShift TSL communication. The value of this 
  parameter can be a relative distinguished name which must be contained in a presented client certificate. Enabling this
  parameter will automatically switch Jolokia into https communication mode. The default CA cert is set to 
  `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` 
  
#### jmx_exporter configuration 

* **AB_JMX_EXPORTER_OPTS** : Configuration to use for `jmx_exporter` (in the format `<port>:<path to config>`)
* **AB_JMX_EXPORTER_PORT** : Port to use for the JMX Exporter. Default: `9779`
* **AB_JMX_EXPORTER_CONFIG** : Path to configuration to use for `jmx_exporter`: Default: `/opt/agent-bond/jmx_exporter_config.json`



### Startup Script /run-java.sh

The default command for this image is `/run-java.sh`. Its purpose it
to fire up Java applications which are provided as fat-jars, including
all dependencies or more classical from a main class, where the
classpath is build up from all jars within a directory.x1

The run script can be influenced by the following environment variables:

* **JAVA_APP_DIR** the directory where the application resides. All paths
  in your application are relative to this directory.
* **JAVA_LIB_DIR** directory holding the Java jar files as well an optional `classpath` 
  file which holds the classpath. Either as a single line classpath (colon separated) or 
  with jar files listed line-by-line. If not set **JAVA_LIB_DIR** is the same 
  as **JAVA_APP_DIR**. 
* **JAVA_OPTIONS** options to add when calling `java`
* **JAVA_MAIN_CLASS** A main class to use as argument for `java`. When
  this environment variable is given, all jar files in `$JAVA_APP_DIR`
  are added to the classpath as well as `$JAVA_LIB_DIR`.
* **JAVA_APP_JAR** A jar file with an appropriate manifest so that it
  can be started with `java -jar` if no `$JAVA_MAIN_CLASS` is set. In all
  cases this jar file is added to the classpath, too.
* **JAVA_APP_NAME** Name to use for the process
* **JAVA_CLASSPATH** the classpath to use. If not given, the script checks 
  for a file `${JAVA_APP_DIR}/classpath` and use its content literally 
  as classpath. If this file doesn't exists all jars in the app dir are 
  added (`classes:${JAVA_APP_DIR}/*`). 
* **JAVA_ENABLE_DEBUG** If set remote debugging will be switched on
* **JAVA_DEBUG_PORT** Port used for remote debugging. Default: 5005

If neither `$JAVA_APP_JAR` nor `$JAVA_MAIN_CLASS` is given,
`$JAVA_APP_DIR` is checked for a single JAR file which is taken as
`$JAVA_APP_JAR`. If no or more then one jar file is found, the script
throws an error. 

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

These variables can be also set in a
shell config file `run-env.sh`, which will be sourced by 
the startup script. This file can be located in the directory where 
this script is located and in `${JAVA_APP_DIR}`, whereas environment 
variables in the latter override the ones in `run-env.sh` from the script 
directory.

This script also checks for a command `run-java-options`. If existant it will be
called and the output is added to the environment variable `$JAVA_OPTIONS`.

Any arguments given during startup are taken over as arguments to the
Java app. 

### Versions:

* Base-Image: **CentOS 7**
* Java: **OpenJDK 7 1.7.0** (Java Development Kit (JDK))
* Agent-Bond: **0.1.3** (Jolokia 2.0.0-M1, jmx_exporter 0.6)
