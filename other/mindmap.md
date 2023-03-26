---
markmap:
  colorFreezeLevel: 0
  maxWidth: 300
---

# Certified Kubernetes Application Developer

## Application Design and Build (20%)
### Define, build and modify container images
- write simple Dockerfile
- build image based on Dockerfile
- push image to repository
- use custom image in a Pod
### Understand Jobs and CronJobs
- define a `Job`
- define a `CronJob`
- define parallelism for a `Job` to execute jobs in parallel
### Understand multi-container Pod design patterns (e.g. sidecar, init and others)
- understand sidecar pattern
- understand adapter pattern
- understand ambassador pattern
- understand `initContainer` pattern
### Utilize persistend and ephemeral volumes
- define a `Volume` within a Pod
- create a `PersistentVolume`
- create a `PersistentVolumeClaim`
- use a `PersistentVolume` within a Pod
- understand how `persistentVolumeReclaimPolicy` works
## Application Deployment (20%)
### Use K8s primitives to implement common deployment strategies(e.g blue/green or canary)
- perform blue/green deployment
- perform canary deployment
- define a deployment with a `Recreate` strategy
### Understand Deployments and how to perform rolling updates
- define a `Deployment` with a `RollingUpdate` strategy
- check `Deployment` rollout status
- perform a rollback
- use a plain text env variable in a `Deployment` definition
### Use the Helm package manager to deploy existing packages
- TODO
- TODO
- TODO
## Application Environment, Configuration and Security (25%)
### Discover and use resources that extend Kubernetes (CRD)
- display available `CustomResourceDefinitions`
- create a `CustomResourceDefinition`
- create `CustomResource` for a given `CustomResourceDefinition`
### Understand authentication, authorization and admission control
- configure external auth method used by `kube-apiserver`
- configure authz method used by `kube-apiserver`
### Understanding and defining resource requirements, limits and quotas
- create a `ResourceQuota` for a given namespace
- define cpu/memory limits for a container in a `Pod`
### Understand ConfigMaps
- create a `ConfigMap` from literal (imperative)
- create a `ConfigMap` from file (imperative)
- mount a `ConfigMap` to a `Deployment` as a volume
- add an env variable from a `ConfigMap` in a `Deployment` definition
### Create & consume Secrets
- add env variable from a `Secret` in a `Deployment` definition

### Understand ServiceAccounts
- create a `ServiceAccount` (imperative)
- bind a `ServiceAccount` to a `Pod`
- create a token for `ServiceAccount`

### Understand SecurityContexts

## Services and Networking (20%)
### Demonstrate basic understanding of NetworkPolicies
- create a `NetworkPolicy` to allow ingress/egress trafic from/to specific pods
- create a `NetworkPolicy` to block all traffic
### Provide and troubleshoot access to applications via services
- create a `NodePort Service`
- create a `ClusterIP Service`
- understand what is `LoadBalancer Service` (?)
### Use Ingress rules to expose applications
- create `Ingress` rule to expose service (imperative)
- create `Ingress` rule to expose multiple services - split by url
- create `Ingress` rule to expose multiple services - split by hostname

## Application observability and maintenance (15%)
### Understand API deprecations
- get list of available API groups using k8s REST API
### Implement probes and health checks
- define `readinessProbe` for a container(httpGet,tcpSocket,exec)
- define `livenessProbe` for a container(httpGet,tcpSocket,exec)
### Use provided tools to monitor K8s applications
- display kubernetes config `kubectl config view`
- use `kubectl proxy` to access k8s API
### Utilize container logs
- display container log(single&multi container pods)
### Debugging in K8s
