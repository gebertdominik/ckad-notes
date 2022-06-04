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



