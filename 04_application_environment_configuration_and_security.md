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

![access_control_overview](/images/006_access_control_overview.svg)
*Soruce: kubernetes.io*

The first step is authentication, where a request needs to go through any authentication module that has been configured. At the authorization step, the request will be checked against existing policies. It will be authorized if the user has the permissions to perform requested actions. Then the request will go through the last step of admission controllers. In general admission controllers will check the actual content of the objects being created and validate them before admitting the request.

In addition, the request reaching the API server over the network is encrypted using TLS. This needs to be configured using SSL certificates.

> **Note**
> 
> If we use `kubeadm`, TLS configuration should be done for us out of the box, otherwise it should be configured manually. Look at [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) or review the [API Server configuration options](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) to set up TLS.


## Authentication

There are three main points to remember with authentication in k8s:

* In its straightforward form, authentication is done with certificates, tokens or basic authentication (i.e. username or password)
* Username are not created by the API, but should be managed by the operating system or an external server
* System accounts are used by processes to acccess the API (read [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) for more)

There are also two types of more advanced authentication mechanisms:

* Webhooks, which can be used to verify bearer tokens
* Connection with an external OpenAPI provider

These are becoming common in environments which already have a SSO or other existing authentication process.

The type of authentication used is defined in the `kube-apiserver` startup options. Below are four examples of a subset of configuration options that would need to be set depending on what choice of authentication mechanism we can choose:

* `--basic-auth-file`
* `--oidc-issuer-url`
* `--token-auth-file`
* `--authorization-webhook-config-file`

One or more Authentication Modules are used: 

* x509 Client Certs
* static token, bearer or bootstrap token
* static password file
* service account
* OpenID connect tokens

Each is tried until successful, and the order is not guaranteed. Anonymous access can also be enabled, otherwise we will get a 401 response. Users are not created by the API, and should be managed by an external system.

> **Note**
>
> To learn more about authentication, see the official [Kubernetes Documentation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
