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


## Authorization

There are three main authorization modes and two globabl Deny/Allow settings. The three main modes are:

* Node
* RBAC
* Webhook

They can be configured as kube-apiserver startup options:

* `--authorization-mode=Node, RBAC`
* `--authorization-mode=Webhook`

The Node authorization is dedicated for kubelet to communicate to the kube-apiserver such that it does not need to be allowed via RBAC. All non-kubelet traffic would then be checked via RBAC.

The authorization modes implement policies to allow requests. Attributes of the requests are checked against the policies (e.g. user, group, namespace, verb).

### RBAC

RBAC stands for Role Based Access Control. RBAC is a method ofregulating access to computer or network resources based on the roles of individual users within organisation. RBAC uses `rbac.authorization.k8s.io` API group.

> **Note**
>
>  RBAC documentation can be found [here](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).


#### Roles

An RBAC `Role` and `ClusterRole` contains rules that represent a set of permissions. Permissions are purely additive - there are no "deny" rules

A `Role` always sets permissions within particular namespace, when we create a Role, we need to specify the namespace it belongs in.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```



`ClusterRole` is a non-namespaced resource. The resources have different names(Role and ClusterRole), because a K8s object always has to be either namespaced or not namespaced; it can't be both.

ClusterRole can be used to:

* define permissions on namespaced resources and be granted access within individual namespace(s)
* define permissions on namespaced resources and be granted access accross all namespaces
* define permissions on cluster-scoped resources

Because `ClusterRoles` are cluster-scoped, we can use them to grant access to:

* cluster-scoped resources (like nodes)
* non-resource endpoints (like `/healthz`)
* namespaced resources (like Pods) across all namespaces

For example we can use `ClusterRole` to allow a particular user to run `kubectl get pods --all-namespaces`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
 ```

#### Bindings

A role binding grands the permissions defined in a role to a user or set of users. It holds a list of subjects(users/groups/service accounts), and a reference to the role being granted.

A `RoleBinding` may reference any `Role` in the same namespace. Alternatively a `RoleBinding` can reference a `ClusterRole` and bind that `ClusterRole` to the namespace of the `RoleBinding`.

Even though the following example refers to a ClusterRole, "dave" will only be able to read Secrests in the "development" namespace, because the Role Binding's namespace is "development"

```
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "dave" to read secrets in the "development" namespace.
# You need to already have a ClusterRole named "secret-reader".
kind: RoleBinding
metadata:
  name: read-secrets
  #
  # The namespace of the RoleBinding determines where the permissions are granted.
  # This only grants permissions within the "development" namespace.
  namespace: development
subjects:
- kind: User
  name: dave # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

`ClusterRoleBinding` is similar to `ClusterRole` but it's purpose is to grant permissions across a qhile cluster.

```
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
 ```

 After we create a binding, we cannot change the `Role` or `ClusterRole` that it refers to.

 The `kubectl auth reconcile` command-line utility creates or updates a manifest file containing RBAC objects, and handles deleting and recreating binding objects if required to change the role they refer to.

#### Referring to resources

RBAC refers to resources using exactly the same name that appears in the URL for the relevant API endpoint. Some k8s APIs involve a subresource, such as the logs for a Pod. A request for a Pod's logs looks like:

`GET /api/v1/namespaces/{namespace}/pods/{name}/log`

In this case , `pods` is the namespaced resource for Pod resources, and `log` is a subresource of `pods`. To represent this in an RBAC role, use a slash to delimit the resource and subresource. To allow a subject to read `pods` and also access the `log` subresource for each of those pods, we write:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-and-pod-logs-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list"]
```

Resources can be also referred by name list. The following example restricts its subject to only `get` or `update` a ConfigMap named `my-configmap`:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: configmap-updater
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing ConfigMap
  # objects is "configmaps"
  resources: ["configmaps"]
  resourceNames: ["my-configmap"]
  verbs: ["update", "get"]
```

> **Warning**
>
>We cannot restrict `create` or `deletecollection` requests by their resource name. For `create`, this limitation is because the name of the new object may not be known at authorization time. If we restrict `list` or `watch` by resourceName, clients must include a metadata.name field selector in their `list` or `watch` request that matches the specified resourceName in order to be authorized. 
>
>For example: `kubectl get configmaps --field-selector=metadata.name=my-configmap`


#### Aggregated ClusterRoles

We can aggregate `ClusterRoles` into one combined `ClusterRole` A controller, running as part of the cluster control plane, watches for `ClusterRole` objects with an `aggregationRule` set. The `aggregationRule` defines a label selector that the controller uses to match other `ClusterRole` objects that should be combined into the `rules` field of this one.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: [] # The control plane automatically fills in the rules
```

If we create a new ClusterRole taht matches the label selector of an existing aggregated ClusterRole, that change triggers adding the new rules into the aggregated ClusterRole. Here is an example that adds rules to the "monitoring" CluserRole, by creating another Cluster role labeled `rbac.example.com/aggregate-to-monitoring: "true"`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-endpoints
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
# When you create the "monitoring-endpoints" ClusterRole,
# the rules below will be added to the "monitoring" ClusterRole.
rules:
- apiGroups: [""]
  resources: ["services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
 ```

### RBAC Process Overview

The basic RBAC flow is to create a certificate for a subject, associate that to a role perhaps in a new namespace, and test. As users and groups are not API objects, we are requiring outside authentication. After generating the certificate against the cluster certificate authority, we can set the credential for the subject using a context, which is a combination of user name, cluster name, authinfo, and possibly namespace. This information can be seen with the `kubectl config get-contexts` command.

Roles can be used to configure an association of `apigroups`, `resources` and `verbs` allowed to them. The user can then be bound to a role limiting what and where thay can work in the cluster.

Here is a short summary of the RBAC process, typically done by the cluster admin:

* Determine or create namespace of the subject
* Create certificate credentials for the subject
* Set the credentials for the user to the namespace using a context
* Create a role for the expected task set
* Bind the user to the role
* Verify the user has limited access.

## Admission Controller