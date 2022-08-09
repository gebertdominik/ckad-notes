# Application observability and maintenance(15%)

## Objectives

* Understand API deprecations
* Implement probes and health checks
* Use provided tools to monitor Kubernetes applications
* Utilize container logs
* Debugging in Kubernetes


# Application Troubleshooting

## Objectives

* Understand and use the troubleshooting flow.
* Monitor applications.
* Review system logs.
* Review agent logs.
* Discuss conformance testing.
* Discuss cluster certification.

## Overview

K8s is sensitive to network issues. Standard Linux tools and processes are the best method for troubleshooting our cluster. If a shell, such as `bash`, is not available in the affected Pod, we should consider deploying another similar pod with a shell, like busybox. DNS configuration files and tools like `dig` are a good place for start. For more difficult challenges, we may need to install other tools like `tcpdump`.

Large and diverse workloads can be difficult to track, so monitoring of usage is essential. Monitoring is about collecting key metrics, such as CPU, memory, and disk usage, and network bandwith on our nodes, as well as monitoring key metrics in our applications. These features have not been ingested into K8s, so exterior tools will be necessary, such as [Prometheus](https://www.prometheus.io/). Prometheus provides a time-series database, as well as integraiton with Grafana for visulalization and dashboards.

Logging activity accross all the nodes is another fature not part of K8s. Using [Fluentd](https://www.fluentd.org/) can be useful data collector for a unified logging layer. Having aggregated logs can help visualize the issues, and provides the ability to search all logs. It's a good place to start when local network troubleshooting does not expose the root cause. 

We are going to review some of the basic `kubectl` commands that we can use to debug what's happening, walk through the basic steps to be able to debug our containers, our pending containers, and also the systems in K8s.

## Basic Troubleshooting Steps

The trouble shooting flow should start with the obvious. It there are errors from the command line, investigate them first. The symptoms of the issue will probably determine the next step to check. Working from the application running inside a container to the cluster as a whole may be a good idea. The application may have a shell we can use, for example:

```
$ kubectl create deployment troubleshoot --image=nginx
$ kubectl exec -ti troubleshoot- -- /bin/sh
```

If the Pod is running, use `kubectl logs pod-name` to view the standard out of the container. Without logs, we may consider deploying a sidecar container in the Pod to generate and handle logging. The next place to check is networking, including DNS, firewalls and general connectivity, using standard Linux commands and tools.

Security settings can also be a challenge. RBAC, covered in the security chapter, provides mandatory or discretionary access control in a granural manner. SELinux and AppArmor are also common issues, especially with network-centric applications.

The issues found with a decoupled system like K8s are similar to those of a traditional datacenter, plus the added layers of K8s controllers:

* Errors from the command line
* Pod logs and state of Pods
* Use shell to troubleshoot Pod DNS and network
* Check node logs for errors, make sure there are enough resources allocated
* RBAC, SELinux or AppArmor for security settings
* API calls to and from controllers to kube-apiserver
* Inter-node network issues, DNS and firewall
* Control plane controllers(control Pods in pending or error state, errors in log files, sufficient resources, etc.).

> **Warning**
> Unlike typical single vendor programs, there is no one organization responsible for updates to k8s. It's because k8s is a high-velocity open source software project.

## Basic Troubleshooting Flow: Pods

Should the error not be seen while staring the Pod, investigate from within the Pod. A flow working from the application running inside a container to the larger cluster as a whole may be a good idea. Some applications will have a shell available to run commands, for example:

```
$ kubectl create deployment tester --image=nginx
$ kubectl exec -ti tester- -- /bin/sh
```

A feature which is still in alpha state, and may change in any way or disappear, is the ability to attach and image to a running process. This is very helpful for testing the existing pod rather than a new pod, with different configuration. Begin by reading through some of the included examples, then try to attach to an existing pod. We will get an error as `--feature-gates=EmphemeralContainers=true` is not set for the kube-apiserver and kube-scheduler by default.

```
$ kubectl alpha debug -h
<output_omitted>

$ kubectl alpha debug registry-6b5bb79c4-jd8fj --image=busybox
error: ephemeral containers are disabled for this cluster
(error from server: "the server could not find the requested resource").
```

Now we can test a different feature. Create a container to understand the view a container has of the node it is running on. Change out the node name, `master` for our master node. Explore the commands we have available.

`kubectl alpha debug node master -it --image=busybox`

On the container:

```
/ # ps
....
31585 root   2:00 /coredns -conf /etc/coredns/Corefile
31591 root   0:21 containerd-shim -namespace moby -workdir /var/lib/containe
31633 root   0:12 /usr/bin/kube-controllers
31670 root   0:05 bird -R -s /var/run/calico/bird.ctl -d -c /etc/calico/conf
31671 root   0:05 bird6 -R -s /var/run/calico/bird6.ctl -d -c /etc/calico/co
31676 root   0:01 containerd-shim -namespace moby -workdir /var/lib/containe
31704 root   0:07 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/con

/ # df
<output_omitted>
```

If the pod is running we can use `kubectl logs pod-name` to view the standard out of the container. Without logs, we may consider deploying a sidecar container in the Pod to generate and handle logging.

The next place to check is networking, including DNS, firewalls, and general connectivity, using standard Linux commands and tools.

For pods without the ability to log on their own, we may have to attach a sidecar container. These can be configured to either stream application logs to their own standard out, or run a logging agent.

Troubleshooting and application begins with the application itself. Is the application behaving as expected? Transient issues are difficult to troubleshoot; difficulties in troubleshooting are also encountered if the issue is intermittent, or if tit concernts occasional slow performance.

Assuming the app is not the issue, we begin by viewing the pods with `kubectl get` commands. Ensure the pods report a status of `Running`. A status of `Pending` typically means a resource is not available from the cluster, such as a properly tainted node, expected sotrage, or enough resources. Other error codes typically require looking at the logs and events of the containers for further troubleshooting. Also we can look for an unusual number of restarts. A container is expected to restart due to several reasons, such as a command argument finishing. If the restarts are not due to that ,it may indicate that the deployed app is having an issue and failing due to panic or probe failure.

View the details of the pod and the status of the containers using the  `kubectl describe pod` command. This will report overall pod status, container configurations and container events. Work through each section looking for a reported error.

Should the reported information not indicate the issue, the next step would be to view any logs of the container, in case there is a misconfiguration or missing resource unknown to the cluster, but requireed by the application. These logs can be seen with the `kubectl logs <pod-name> <container-name>` command.

## Basic Troubleshooting Flow: Node and Security