# Kubernetes architecture

Introduction to kubernetes architecture - it's not directly mapped to the exam curriculum, but it's required to understand some basics about the architecture.

## Objectives

* Learn  common terms related to k8s
* Discuss history of k8s
* Understand what are control plane node components and worker node components
* Learn about Container Network Interface (CNI) configuration and Network Plugin

## What is Kubernetes?

According to k8s website it's an "open-source system for automating deployment, scaling, and management of contenerized applications". His precursor is Borg project, developed for 15 years by Google.

Other definition says that Kubernetes is an orchestration system to deploy and manage containers.

Kubernetes is written in Go Language - some claim it incorporates the best, whicle some claim the worst parts of C++, Python and Java

## Challenges

We need a CI pipeline to build, test, and verify our container images. Then we need an infrastucture on which we can run our containers and watch over them when things fail and self-heal. We also must be able to perform rollbacks, rolling updates and tear down resources when no longer needed.

All of those require flexible easy-to-manage network and storage. As containers are lunched on any worker node, the network must join the resource to other existing containers, while still keeping the trafic secure from otherrs. We also need a mainainable storage structure.

## Architecture

![kubernetes_architecture.svg](/images/001_kubernetes_architecture.png)

Kubernetes in its simplest form is made of one or more central managers(masters) and worker nodes. For testing puposes it can be combined to a single node. The manager runs an API server, scheduler, operators and a datastore to keep the state of the cluster container settings, and the networking configuration.

K8s exposes the API via the API server, which can be reached using local client - `kubectl`. The kube-scheduler sees the API request for running a new container and finds a suitable node to run that container. 

Each node in the cluster runs two containers:

* kubelet, which receives spec information for container configuration, downloads and manages any resources and works with the container engine on the local node to ensure the container runs or is restarted upon failure.
* kube-proxy, creates and manages local firewall rules and networking configuration to expose containers on the network.

Containers in k8s are not managed individually - they are part of a **Pod**. A Pod consists at least one container. They are all share an IP address, namespace and storage. Typically one container in a Pod runs an application while other containers support the primary application.

Orchestration is managed through **operators** and **controllers**. Each operator uses the kube-apiserver for a particular object state, modifying the object until the declared state matches the current status.

The default operator for container is **Deployment**. A Deployment deploys and manages a different operator called **ReplicaSet** A ReplicaSet is an operator which deploys multiple pods, each with the same spec information. These are called **replicas**.

The Kubernetes architecture is made up of many operators such as **Jobs** and **CronJobs** to handle single or recurring tasks, or custom resource definitions and purpose-built operators.

Kubernetes probides several API objects which can be used to deploy pods, other than just ensuring a certain number of replicas is running somewhere. A **DeamonSet** can be used to ansure that a single pod is deployed on every node. These are often used for logging metrics and security pods. A **StatefulSet** can be used to deploy pods in a particular order, such taht following pods are deployed only if previous pod report a ready statys. It can be used for legacy applications which are not cloud-friendly.

To make cluster management easier, we can use **labels** - strings which become part of the object metadata. They can be use when selecting objects, when don't know the pod name. Nodes can have **taints**, and arbitrary string in the node metadata, to inform the scheduler on Pod assignments used along with toleration in Pod metadata, indicating it should be scheduled on a node with the particular taint.

There is also space in metadata for **annotations**, which remain with the object, but cannot be used as a selector. However, they could be leveraged by other objects or Pods.

Often multiple users and teams share access to one or more clusters. This is referred as MULTI-TENANCY. Some form of isolation is necessary in this case. It can be achieved by using:

* namespace - a segregation of resources, upon witch resource quotas and permissions can be applied. K8s objects may be created in a namespace or cluster-scoped. Users can be limited by the object verbs allowed per namespace. Also **LimitRange** admission controller constraints resource usage in taht namespace. Two objects cannot have the same name in the same namespace.
* context - combination of user, cluster name and namespace. A convinient way to switch between combinations of permissions and restrictions. This information is referenced from `~/.kube/config`
* Resource Limits - a way to limit the amount of resources consumed by a pod or to request a minimum amount of resources reserved but not necessarily consumed by a pod. Limits can also be set per-namespaces, which have priority over those in the PodSpec
* Pod Security Policies - Deprecated. It was a policy to limit the ability of pods to elevate permissions or modify the node upon which they are scheduled. This wide-ranging limitation may prevent apod from operating properly. This was replaced with Pod Security Admission. Some have gone towards Open Policy Agent, or other tools instead.
* Pod Security Admission - a beta feature to restrict pod behaviour in an easy-to-implement and easy-to-understand manner, applied at the namespace level when a pod is created. These will leverage three profiles: Privilaged, Baseline, and Restricted policies.
* Network Policies - the ability to have an inside-the cluster firewall. Ingress and Egress traffic can be limited according to namespaces and labels as well as typical network traffic characteristics.

## Control Plane Nodes

K8s Mmaster runs various server and manager processes for the cluster. Among the componentd of the master node are the kube-apiserver, the kube-scheduler and the etcd database. As the software has matured, new components have been created to handle dedicated needs, such as the cloud-controller-manager. It handles tasks, once handled by the kube-controller-manager, to interact with other tools such as Rancher or DigitalOcean for third-party cluster management and reporting.

There are several add-ons which have become essential to a typical production cluster, such as DNS services. Others are third-party solutions where K8s has not yet developed a local component, such as cluster-level logging and resource monitoring.


