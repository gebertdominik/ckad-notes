# Application Environment, Configuration and Security(25%)

## Objectives

* Discover and use resources that extend Kubernetes (CRD)
* Understand authentication, authorization and admission control
* Understanding and defining resource requirements, limits and quotas
* Understand ConfigMaps
* Create & consume Secrets
* Understand ServiceAccounts
* Understand SecurityContext


# Security

## Objectives

* Explain the API flow of API requests
* Configure authorization rules
* Examine authentication policies
* Restric network traffic with network policies

## Accessing the API

To access the API to perform any action in k8s, we need to go through three main steps:

* Authentication (Certificate or webhook)
* Authorization (RBAC or Webhook)
* Admission Controls

![access_contro_overview](/images/005_access_contro_overview.svg)
*Soruce: kubernetes.io*

The first step is authentication, where a request needs to go through any authentication module that has been configured. At the authorization step, the request will be checked against existing policies. It will be authorized if the user has the permissions to perform requested actions. Then the request will go through the last step of admission controllers. In general admission controllers will check the actual content of the objects being created and validate them before admitting the request.

In addition, the request reaching the API server over the network is encrypted using TLS. This needs to be configured using SSL certificates.

> **Note**
> 
> If we use `kubeadm`, TLS configuration should be done for us out of the box, otherwise it should be configured manually. Look at [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) or review the [API Server configuration options](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) to set up TLS.