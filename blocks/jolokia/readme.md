#### Jolokia configuration

* **AB_JOLOKIA_OFF** : If set disables activation of Jolokia (i.e. echos an empty value). By default, Jolokia is enabled.
* **AB_JOLOKIA_CONFIG** : If set uses this file (including path) as Jolokia JVM agent properties (as described 
  in Jolokia's [reference manual](http://www.jolokia.org/reference/html/agents.html#agents-jvm)). If not set, 
  the `/opt/jolokia/etc/jolokia.properties` will be created using the settings as defined in this document, otherwise
  the reset of the settings in this document are ignored.
* **AB_JOLOKIA_HOST** : Host address to bind to (Default: `0.0.0.0`)
* **AB_JOLOKIA_PORT** : Port to use (Default: `8778`)
* **AB_JOLOKIA_USER** : User for basic authentication. Defaults to 'jolokia'
* **AB_JOLOKIA_PASSWORD** : Password for basic authentication. By default authentication is switched off.
* **AB_JOLOKIA_PASSWORD_RANDOM** : Should a random AB_JOLOKIA_PASSWORD be generated? Generated value will be written to `/opt/jolokia/etc/jolokia.pw`
* **AB_JOLOKIA_HTTPS** : Switch on secure communication with https. By default self signed server certificates are generated
  if no `serverCert` configuration is given in `AB_JOLOKIA_OPTS`
* **AB_JOLOKIA_ID** : Agent ID to use (`$HOSTNAME` by default, which is the container id)
* **AB_JOLOKIA_DISCOVERY_ENABLED** : Enable Jolokia discovery.  Defaults to false.
* **AB_JOLOKIA_OPTS**  : Additional options to be appended to the agent configuration. They should be given in the format 
  "key=value,key=value,..."

Some options for integration in various environments:

* **AB_JOLOKIA_AUTH_OPENSHIFT** : Switch on client authentication for OpenShift TSL communication. The value of this 
  parameter can be a relative distinguished name which must be contained in a presented client certificate. Enabling this
  parameter will automatically switch Jolokia into https communication mode. The default CA cert is set to 
  `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` 

