# Namespace Best Practices

## Table of Contents

[Namespace](#namespace)
- Intermediate
    + [Don’t Use Default Namespace](#dont-use-default-namespace)
    + [Naming Convention](#naming-convention)
    + [Labels](#labels)
- Moderate
    + [Limit Resource Usages](#limit-resource-usages)
- Experienced
    + [When To Use Multiple Namespaces](#when-to-use-multiple-namespaces)
    + [Should Have RBAC Configured If Multiple Namespaces](#should-have-rbac-configured-if-multiple-namespaces)

---
---

## Namespace

> Namespaces are a way for isolating group of resouces within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. In simple words namespaces are a way to divide cluster resources between many users spread across multiple teams, or projects. There are multiple benefits to using namespaces, but using them improperly can also cause difficulties with your Kubernetes projects. In this guide, you’ll learn some best practices to follow for creating and using namespaces.

By default kubernets comes with below namespaces out of the box.

1. default
2. kube-system, 
3. kube-node-lease
4. kube-public

Here is an example of how to create namespace.

```
kubectl create namespace <<NAMESPACE>>
```
---

### Don’t Use Default Namespace

For a production cluster, consider not using <mark>default</mark> namespace. Instead, make other namespaces and use those.

### Naming Convention

- Avoid creating namespaces with the prefix <mark>kube-</mark>, since it is reserved for kubernetes system namespaces.
- They should only contains alphanumeric characters and hyphens.
- The alpha characters can only be lower-case, and names cannot start with hyphens

### Labels

Labels are key-value pairs used to describe an application. A common set of labels allows tools to work interoperably. So whenever creationg namespace we must include labels into it. Kubernetes recommends using a set of standard labels to describe an application. These can be helpful in documenting and organizing your application. 
Read [this](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/) guide to understand Recommended labels and also check [Well-Known labels](https://kubernetes.io/docs/reference/labels-annotations-taints/).

### Limit Resource Usages

Containers without limits can lead to resource contention with other containers and unoptimized consumption of computing resources. Kubernetes has two features for constraining resource utilisation: <b>ResourceQuota</b> and <b>LimitRange</b>.

With the <b>LimitRange</b> object, you can define default values for resource requests and limits for individual containers inside namespaces.
With the <b>ResourceQuotas</b>, you can limit the total resource consumption of all containers inside a Namespace. You can also set quotas for other Kubernetes objects such as the number of Pods in the current namespace.

Read [official document](https://kubernetes.io/docs/concepts/policy/resource-quotas/) for more information.

### When To Use Multiple Namespaces

For cluster with a few to tens of users, you should not need to create or think about namespaces at all. If the workgroups are separated, users can work together with relative freedom while isolated from unrelated groups. This helps ensuring that the resources and constraints of one project/Team don’t affect any others.

### Should Have RBAC Configured If Multiple Namespaces

Enhancing role-based access controls (RBAC) by limiting users and processes to certain namespaces. RBAC can work with either the <b>ClusterRole</b> resource type, which is scoped to the entire cluster, or the <b>Role</b> resource type, which is scoped to a specific namespace. This means that you can limit the resources a user is allowed to access, either globally or within a particular namespace.