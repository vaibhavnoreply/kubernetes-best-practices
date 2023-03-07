# Monitoring Best Practices

## Table of Contents

[Monitoring](#monitoring)
- Intermediate
    + [Don't use Metrics Server](#dont-use-metrics-server)
    + [Monitor Nodes, Pods, Network and cluster health using dashboards](#monitor-nodes-pods-network-and-cluster-health-using-dashboards)
- Moderate
    + [Deploy Prometheus or Grafana to enable monitoring capabilities with data persistence](#deploy-prometheus-or-grafana-to-enable-monitoring-capabilities-with-data-persistence)
- Experienced
    + [Tune Prometheus and Grafana servers based on retention period and amount of metrics](#tune-prometheus-and-grafana-servers-based-on-retention-period-and-amount-of-metrics)
---
---

## Monitoring
---

#### Don't use Metrics Server

Metrics Server is not meant for non-autoscaling purposes. For example, don't use it to forward metrics to monitoring solutions, or as a source of monitoring solution metrics. In such cases please collect metrics from Kubelet /metrics/resource endpoint directly.

When you need:

- Non-Kubernetes clusters
- An accurate source of resource usage metrics
- Horizontal autoscaling based on other resources than CPU/Memory

#### Monitor Nodes, Pods, Network and cluster health using dashboards. 

Deploy Prometheus server, metrics exporters, setup kube-state-metrics, pull, scrape and collect metrics, configure alerts with Alertmanager and dashboards with Grafana.

Read [this](https://sysdig.com/blog/kubernetes-monitoring-prometheus) for more details.

#### Deploy Prometheus or Grafana to enable monitoring capabilities with data persistence

Deploy Prometheus server, metrics exporters, setup kube-state-metrics, pull, scrape and collect metrics, configure alerts with Alertmanager and dashboards with Grafana.

#### Tune Prometheus and Grafana servers based on retention period and amount of metrics

The parameter for the frequency to remove old data. The default value is 24 hours (24h).