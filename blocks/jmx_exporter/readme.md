#### jmx_exporter Configuration

* **AB_OFF** and **AB_PROMETHEUS_OFF**: If set to any value except 'false', disable Prometheus export
* **AB_PROMETHEUS_PORT** : Port to expose Prometheus (default: 9779)
* **AB_PROMETHEUS_HOST** : Host address to listen on for exporting Prometheus metrics. Otherwise the exporter listens on any IP
* **AB_PROMETHEUS_CONFIG** : Path to the Prometheus configuration file (default: `/opt/prometheus/prometheus-config.yml`
