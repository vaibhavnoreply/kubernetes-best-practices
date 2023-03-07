# Application deployment Best Practices

## Table of Contents

[Application deployment](#application-deployment)
- Intermediate
    + [Bind pods to deployment or replicaset (Avoid using naked pod)](#bind-pods-to-deployment-or-replicaset-avoid-using-naked-pod)
    + [Don't use latest tags](#dont-use-latest-tags)
    + [One process per container](#one-process-per-container)
    + [Create services before their workloads](#create-services-before-their-workloads)
    + [Configure a liveness probe](#configure-a-liveness-probe)
    + [Configure a readiness probe](#configure-a-readiness-probe)
    + [Labels](#labels)
- Moderate
    + [Run more than one replica for deployments](#run-more-than-one-replica-for-deployments)
    + [Avoid Pods being placed into a single node](#avoid-pods-being-placed-into-a-single-node)
    + [Limit Resource Usages](#limit-resource-usages)
    + [Set CPU request to 1 CPU or below](#set-cpu-request-to-1-cpu-or-below)
    + [Disable CPU limits - Unless you have a good use case](#disable-cpu-limits-unless-you-have-a-good-use-case)
    + [Mount secrets as volume not as ENV variables](#mount-secrets-as-volume-not-as-env-variables)
    + [Use configMaps to store application configurations and env variables](#use-configmaps-to-store-application-configurations-and-env-variables)
- Experienced
    + [If using multiple replica of deployments avoid using local state](#if-using-multiple-replica-of-deployments-avoid-using-local-state)
    + [Allow deploying containers only from known registries](#allow-deploying-containers-only-from-known-registries)
    + [Try running container as non root user](#try-running-container-as-non-root-user)
    + [Choose deployment strategy wisely](#choose-deployment-strategy-wisely)
---
---

## Application deployment
---

#### Bind pods to deployment or replicaset (Avoid using naked pod)

Pods are the fundamental Kubernetes building block for your container and now you hear that you shouldn't use Pods directly but through an abstraction such as a Deployment or replicaset.

If you deploy a Pod directly to your Kubernetes cluster, your container(s) will run, but nothing takes care of its lifecycle. Once a node goes down, capacity on the current node is needed, etc the Pod will get lost forever. 

Thats the point where building blocks such as ReplicaSet and Deployment come into play. A ReplicaSet acts as a supervisor to the Pods it watches and recreates Pods that don´t exist anymore. Deployments are an even higher abstraction and create and manage ReplicaSets to enable the developer to use a declarative state instead of imperative commands (e.g. kubectl rolling-update). The real advantage is that Deployments will automatically do rolling-updates and always ensure a given target state instead of having to deal with imperative changes.

Read [reference document](https://gist.github.com/StevenACoffman/3e626ada8ef8240a9eccdcc435e4a174) for more information.

#### Don't use latest tags

When creating Kubernetes Pods, it is for certain reasons not advisable to use `latest` tags on Container Images.

The reason of not using latest tags is that once we create kubernetes pod it assigns `imagePullPolicy` to it. By default this will set to `IfNotPresent` which means that the container runtime will only pull the image if it is not present on the node the Pod was assigned to. Once you use `latest` as an image tag this default behavior switches to `Always` resulting in the runtime pulling the image every time it starts up a container using that image in a Pod.

There are two really important reasons why this is really bad thing to do:
- You loose control over which exact code is running in your system
- Rolling-Updates/Rollbacks are not possible anymore

Read [reference document](https://gist.github.com/StevenACoffman/3e626ada8ef8240a9eccdcc435e4a174) for more information.

#### One process per container

There are very practical reasons why you may want to consider following "one process per container" as a rule of thumb:

- Scaling containers horizontally is much easier.
- Having a single function per container allows the container to be easily re-used for other projects or purposes.
- It also makes it more portable and predictable for devs to pull down a component from production to troubleshoot locally rather than an entire application environment.
- Patching/upgrades (both the OS and the application) can be done in a more isolated and controlled manner. 
- Splitting functions out to multiple containers allows more flexibility from a security and isolation perspective.

#### Create services before their workloads

When you have a Pod that needs to access a Service, and you are using the environment variable method to publish the port and cluster IP to the client Pods, you must create the Service before the client Pods come into existence. Otherwise, those client Pods won't have their environment variables populated.

If you only use DNS to discover the cluster IP for a Service, you don't need to worry about this ordering issue.

#### Configure a liveness probe

The Liveness probe is designed to restart your container when it's stuck. Kubernetes performs a health check to ensure the application is responsive and running as intended. In the event the `livenessProbe` fails, the kubelet default policy `restarts` the container to bring it back-up.

#### Configure a readiness probe

Kubernetes checks if the application is ready to start serving traffic before allowing traffic to a pod. This essentially indicates whether a pod is available to accept traffic and respond to requests.

#### Labels

Labels are key-value pairs used to describe an application. A common set of labels allows tools to work interoperably. So whenever creationg namespace we must include labels into it. Kubernetes recommends using a set of standard labels to describe an application. These can be helpful in documenting and organizing your application. Read [this](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/) guide to understand Recommended labels.

#### Run more than one replica for deployments

When you run two or more replicas per service, you are increasing the availability of the containerized service because you are not relying on a single pod to do all of the work. Furthermore, scaling horizontally by adding pods allows your service to scale and handle larger loads.

#### Avoid Pods being placed into a single node

Even if you run several copies of your Pods, there are no guarantees that losing a node won't take down your service. Consider the following scenario: you have 11 replicas on a single cluster node. If the node is made unavailable, the 11 replicas are lost, and you have downtime.

We should apply [anti-affinity rules](https://cloudmark.github.io/Node-Management-In-GKE/#pod-anti-affinity-rules) so that pods are spread in all the nodes of your cluster.

#### Limit Resource Usages

Resource limits are used to constrain how much CPU and memory your containers can utilise and are set using the resources property of a containerSpec. 

With the <b>LimitRange</b> object, you can define default values for resource requests and limits for individual containers inside namespaces.
With the <b>ResourceQuotas</b>, you can limit the total resource consumption of all containers inside a Namespace. You can also set quotas for other Kubernetes objects such as the number of Pods in the current namespace.

#### Set CPU request to 1 CPU or below

Unless you have computational intensive job, it is recommended to set the request to 1 cpu or below.

For more information watch [this](https://www.youtube.com/watch?v=xjpHggHKm78) video.

#### Disable CPU limits - Unless you have a good use case

CPU is measured as CPU timeunits per timeunit.
- If you have 1 thread, you can't consume more than 1 CPU second per second.
- If you have 2 threads, you can consume 1 CPU second in 0.5 seconds.
- 8 threads can consume 1 CPU second in 0.125 seconds.

If you're not sure about what's the best settings for your app, it's better not to set the CPU limits.

#### Mount secrets as volume not as ENV variables

The content of Secret resources should be mounted into containers as volumes rather than passed in as environment variables. This is to prevent that the secret values appear in the command that was used to start the container, which may be inspected by individuals that shouldn't have access to the secret values.

#### Use configMaps to store application configurations and env variables

In Kubernetes, the configuration can be saved in ConfigMaps, which can then be mounted into containers as volumes are passed in as environment variables. Save only non-sensitive configuration in ConfigMaps. For sensitive information (such as credentials), use the Secret resource.

#### If using multiple replica of deployments avoid using local state

Storing persistent data in a container's local filesystem prevents the encompassing Pod from being scaled horizontally. Instead, any persistent information should be saved at a central place outside the Pods. For example, in a PersistentVolume in the cluster, or even better in some storage service outside the cluster.

#### Allow deploying containers only from known registries

One of the most common custom policies that you might want to consider is to restrict the images that can be deployed in your cluster.

Read [this](https://blog.openpolicyagent.org/securing-the-kubernetes-api-with-open-policy-agent-ce93af0552c3#3c6e) to get more details.

#### Try running container as non root user

Running your containers as non-root gives you an extra layer of security. By default, Docker containers are run as root, but this allows for unrestricted container activities.

```
RUN groupadd -r nodejs
RUN useradd -m -r -g nodejs nodejs
USER nodejs
```

```
apiVersion: v1
kind: Pod
metadata:
name: hello-world
spec:
containers:
# specification of the pod’s containers
# ...
securityContext:
 runAsNonRoot: true
```

#### Choose deployment strategy wisely

Regarding Replicaset deployment choose one of these deployment strategy based on your need.

- RollingUpdate:
    - Default strategy
    - Pro: No Downtime.
    - Cons: Deployment can be time-consuming and there is no traffic control between versions.
- Recreate:
    - Pro: Remove previous problematic versions quickly.
    - Cons: Downtime may be relevant depending on the cold start of applications.      
- Blue-Green:
    - A blue/green deployment duplicates the environment with two parallel versions.
    - It's a great way to reduce service downtime and ensure all traffic is transferred immediately.
- Canary:
    - Canary deployment is a relevant way to test new versions without driving all the traffic right away.
    - The idea is to separate a small part of customers for the new version and gradually increase it until the entire flow is validated or discarded. 