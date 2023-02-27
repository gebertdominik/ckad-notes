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
* Restrict network traffic with network policies

## Accessing the API

To access the API to perform any action in k8s, we need to go through three main steps:

* Authentication (Certificate or webhook)
* Authorization (RBAC or Webhook) - check against existing policies
* Admission Controls - check the actual content of the objects being created and validate them before admitting the request

![access_control_overview](./images/006_access_control_overview.svg)
*Soruce: kubernetes.io*

In addition, the request reaching the API server over the network is encrypted using TLS. This needs to be configured using SSL certificates.

> **Note**
> 
> If we use `kubeadm`, TLS configuration should be done for us out of the box, otherwise it should be configured manually. Look at [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) or review the [API Server configuration options](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) to set up TLS.


## Authentication

There are three main points to remember with authentication in k8s:

* In its straightforward form, authentication is done with certificates, tokens or basic authentication (i.e. username or password)
* Username are not created by the API, and should be managed by the operating system or an external server
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

There are three main authorization modes and two global Deny/Allow settings. The three main modes are:

* Node
* RBAC
* Webhook

They can be configured as kube-apiserver startup options:

* `--authorization-mode=Node, RBAC`
* `--authorization-mode=Webhook`

The Node authorization is dedicated for kubelet to communicate to the kube-apiserver such that it does not need to be allowed via RBAC. All non-kubelet traffic would then be checked via RBAC.

The authorization modes implement policies to allow requests. Attributes of the requests are checked against the policies (e.g. user, group, namespace, verb).

### RBAC

RBAC stands for Role Based Access Control. RBAC is a method of regulating access to computer or network resources based on the roles of individual users within organisation. RBAC uses `rbac.authorization.k8s.io` API group. RBAC documentation can be found [here](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).


#### Roles

An RBAC `Role` and `ClusterRole` contains rules that represent a set of permissions. Permissions are purely additive - there are no "deny" rules

A `Role` always sets permissions within particular namespace, when we create a Role, we need to specify the namespace it belongs in.

```yaml
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

```yaml
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

A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects(users/groups/service accounts), and a reference to the role being granted.

A `RoleBinding` may reference any `Role` in the same namespace. Alternatively a `RoleBinding` can reference a `ClusterRole` and bind that `ClusterRole` to the namespace of the `RoleBinding`.

Even though the following example refers to a ClusterRole, "dave" will only be able to read Secrests in the "development" namespace, because the Role Binding's namespace is "development"

```yaml
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

`ClusterRoleBinding` is similar to `ClusterRole` but it's purpose is to grant permissions across a whole cluster.

```yaml
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

```yaml
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

```yaml
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

```yaml
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

If we create a new ClusterRole that matches the label selector of an existing aggregated ClusterRole, that change triggers adding the new rules into the aggregated ClusterRole. Here is an example that adds rules to the "monitoring" CluserRole, by creating another Cluster role labeled `rbac.example.com/aggregate-to-monitoring: "true"`

```yaml
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

Admissions controllers can access the conent of the objects being created by the requests. They can modify content, validate it, and potentially deny request.

Admission controllers are needed for certain features to work properly. Controllers have been added as K8s matured. Started with 1.13.1 release of the `kube-apiserver`, the admission controllers are now compiled into the binary, instead of a list passed during execution. To enable or disable, we can pass the following options, changing out the plugins we want to enable or disable:

* `--enable-admission-plugins=NamespaceLifecycle,LimitRanger`
* `--disable-admission-plugins=PodNodeSelector`

Controllers becoming more common are `MutatingAdmissionWebhook` and `ValidatingAdmissionWebhook` which will allow the dynamic modification of the API request, providing greater flexibility. These calls reference an exterior service, such as OPA, and wait for a return API call. Each admission controller functionality is explained in the [documentation](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/).

## Security Contexts

Pods and containers can be given security constraints to limit what processes running in the containers can do(for example we can limit the Linux capabilities).

**Clusters instaling using kubeadm allow pods any possible elevation in privilege by default.** For example a pod could control the nodes networking config, override root etc. These abilities are almost always limited by cluster administrators.

This security limitation is called a security context. It can be defined for the entire pod, or for each container.

For example, if we want to enforce policy that containters cannot run their process as the root user, we can add a pod security context like:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - image: nginx
    name: nginx
```

Then when we create this Pod, we will see a warning that the container is trying to run as root, which is not allowed. Hence, the pod will never run:

```
$ kubectl get pods
NAME   READY  STATUS                                                 RESTARTS  AGE
nginx  0/1    container has runAsNonRoot and image will run as root  0         10s
```

> **Note**
>
>More info about Security Contexts can be found in the [Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

## Pod Security Policies (PSPs)

> **Warning**
>
> PodSecurityPolicy was [deprecated](https://kubernetes.io/blog/2021/04/08/kubernetes-1-21-release-announcement/#podsecuritypolicy-deprecation) in Kubernetes v1.21, and removed from Kubernetes in v1.25. More about this can be found in the [PodSecurityPolicy Deprecation](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/) blog post. 



## Pod Security Standards

There are 3 policies to limit what a pod is allowed to do. Those policies are cumulative. The namespace is given the appropriate label for each policy. New pods will then be restricted. Existing pods would not be changed by an edit to the namespace.

> **Note**
>
>Details can be found in [Kubernetes documentation](https://kubernetes.io/docs/concepts/security/pod-security-standards/#privileged)

### Privileged

No restrictions from this policy.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: no-restrictions-namespace
  labels:
    pod-security.kubernetes.io/enforce: privileged
    pod-security.kubernetes.io/enforce-version: latest
```

### Baseline

Minimal restrictions. Does not allow known privilege escalations.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: baseline-namespace
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: baseline
    pod-security.kubernetes.io/warn-version: latest
```

### Restricted

Most restricted policy. Follows current pod hardening best practicies.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-restricted-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
```

## Network Security Polcies

Network Security Polcies are usually set up by administrators.

* By default, all pods can reach each other. 
* All ingress and egress traffic is allowed - this has been a high-level networking requirement in K8s. 
* network isolation can be configured and traffic to/from pods can be blocked using `NetworkPolicy`
* As all traffic is allowed, we may want to implement a policy that drops all traffic, then, other policies which allow desired ingress and egress trafic.

The `spec` of the policy can narrow down the effect to a particular namespace, which can be handy. Further settings include a `podSelector`, or label, to narrow down which Pods are affected. Further ingress and egress settings declare traffic to and from IP addresses and ports.

Not all network providers support `NetworkPolicies` kind. A non-exhaustive list of providers with support includes Calico, ROmana, Cilium, Kube-router and WeaveNet.

In previous versions of K8s, there was a requirement to annotate a namespace as part of network isolation, specifically the `net.beta.kubernetes.io/network-policy=value`. Some network plugins may still require this setting.

The use of policies has become stable, noted with the `v1` `apiVersion`. The example below narrows down the policy to affect the default namespace.

Only Pods with the label of `role: db` will be affected by the policy below, and the policy has both Ingress and Egress settings.

The `ingress` setting includes a `172.17` network, with a smaller range of `171.17.1.0` IPs being excluded from this traffic.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-egress-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
  - namespaceSelector:
      matchLabels:
        project: myproject
  - podSelector:
      matchLabels:
        role: frontend
  ports:
  - protocol: TCP
    port: 6379
egress:
- to:
  - ipBlock:
      cidr: 10.0.0.0/24
  ports:
  - protocol: TCP
    port: 5978
```

These rules change the namespace for the following settings to be labeled `project: myproject`. The affected Pods also would need to match the label `role: frontend`. Finally, TCP traffic on port 6379 would be allowed from these Pods.

The egress rule have the `to` settings, in this case the `10.0.0.0/24` range TCP traffic to port 5978.

The use of empty ingress or egress rules denies all type of traffic for the included Pods, through this is not suggested. Use another dedicated `NetworkPolicy` instead.

> **Note**
>
> There can also be complex `matchExpressions` statements in the spec, but this may change as `NetworkPolicy` matures.
>```
>podSelector:
>  matchExpressions:
>    - {key: inns, operator: In, values: ["yes"]}
>```

> **Note**
>
> More network policies, can be found on [Github](https://github.com/ahmetb/kubernetes-network-policy-recipes)

### Default Policy Example

The empty braces will match all Pods not selected by other `NetworkPolicy` and will not allow ingress traffic. Egress traffic would be unaffected by this policy:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

With the potential for complex ingress and egress rules, it may be helpful to create multiple objects which include simple isolation rules and use easy to understand names and labels.

Some network plugins, such as WeaveNet, may requiree annottation of the Namespace. The following shows the setting of a `DefaultDeny` for the `myns` namespace:

```
kind: Namespace
apiVersion: v1
metadata:
  name: myns
  annotations:
    net.beta.kubernetes.io/network-policy: |
     {
        "ingress": {
          "isolation": "DefaultDeny"
        }
     }
```