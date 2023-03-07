# Scaling Best Practices

## Table of Contents

[Scaling](#scaling)
- Intermediate
    + [Should have metrics server installed](#should-have-metrics-server-installed)
    + [Containers should not store any state in local filesystem](#containers-should-not-store-any-state-in-local-filesystem)
- Moderate
    + [Use the Horizontal Pod Autoscaler for apps with variable usage patterns](#use-the-horizontal-pod-autoscaler-for-apps-with-variable-usage-patterns)
    + [Don't use the Vertical Pod Autoscaler while it's still in beta](#dont-use-the-vertical-pod-autoscaler-while-its-still-in-beta)
---
---

## Scaling
---

#### Should have metrics server installed

Metrics Server collects resource metrics from Kubelet and exposes them in Kubernetes apiserver through Metrics API for use by `Horizontal Pod Autoscaler` and `Vertical Pod Autoscaler`.

Metrics Server offers:
- A single deployment that works on most clusters
- Fast autoscaling, collecting metrics every 15 seconds.
- Resource efficiency, using 1 mili core of CPU and 2 MB of memory for each node in a cluster.
- Scalable support up to 5,000 node clusters.

#### Containers should not store any state in local filesystem

Containers have a local filesystem and you might be tempted to use it for persisting data. However, storing persistent data in a container's local filesystem prevents the encompassing Pod from being scaled horizontally (that is, by adding or removing replicas of the Pod).

#### Use the Horizontal Pod Autoscaler for apps with variable usage patterns

The [Horizontal Pod Autoscaler (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) is a built-in Kubernetes feature that monitors your application and automatically adds or removes Pod replicas based on the current usage. Configuring the HPA allows your app to stay available and responsive under any traffic conditions, including unexpected spikes.

#### Don't use the Vertical Pod Autoscaler while it's still in beta

This can be useful for scaling applications that can't be scaled horizontally. However, the VPA is currently in beta and it has some known [limitations](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler#limitations-of-beta-version) (for example, scaling a Pod by changing its resource requirements, requires the Pod to be killed and restarted).